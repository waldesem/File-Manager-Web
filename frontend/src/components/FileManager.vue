<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { onBeforeMount, defineAsyncComponent } from "vue";

const ManagerForm = defineAsyncComponent(
  () => import("../components/ManagerForm.vue")
);

const server = "http://127.0.0.1:3000";
// const server = "";

onBeforeMount( async () => {
  await fileManager.value.getFoldersFiles();
});

const fileManager = ref({
  path: ["/"],
  folders: Array<string>(),
  files: Array<string>(),
  selected: Array<string>(),
  copied: Array<string>(),
  select: false,
  action: "",
  form: "",
  item: "",
  server: "",

  
  getFoldersFiles: async function () {
    try {
      const response = await axios.get(`${server}/manager`);
      const { path, dirs, files } = response.data;
      this.assignValue(path, dirs, files);
      this.clearValue();
    } catch (error) {
      console.error(error);
    }
  },

  openFolder: async function (flag: string = "open", item: string = "") {
    try {
      const response = await axios.post(
        `${server}/manager/${flag}`,
        {
          path: this.path,
          item: item,
        }
      );
      const { path, dirs, files } = response.data;

      this.assignValue(path, dirs, files);
    } catch (error) {
      console.error(error);
    }
  },

  openFile: async function (file: string) {
    try {
      const response = await axios.post(
        `${server}/manager/download`,
        {
          path: this.path,
          item: file,
        },
        { responseType: "blob" }
      );

      const fileName = file.split("/").pop();
      const fileExtension = fileName?.split(".").pop();

      this.item = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement("a");
      link.href = this.item;
      link.download = `${fileName}.${fileExtension}`;
      link.dispatchEvent(new MouseEvent("click"));
    } catch (error) {
      console.error(error);
    }
  },

  updateItem: async function () {
    try {
      const response =
        this.action === "create"
          ? await axios.post(
              `${server}/manager/${this.action}`,
              {
                path: this.path,
              }
            )
          : await axios.post(
              `${server}/manager/${this.action}`,
              {
                path: this.path,
                old: this.selected[0],
                new: this.form,
              }
            );
      const { path, dirs, files } = response.data;
      this.assignValue(path, dirs, files);
      this.clearValue();
    } catch (error) {
      console.error(error);
    }
  },

  copyItem: async function () {
    if (this.selected.length) {
      try {
        const response = await axios.post(
          `${server}/manager/${this.action}`,
          {
            path: this.path,
            old: this.copied,
            new: this.selected,
          }
        );
        const { path, dirs, files } = response.data;
        this.assignValue(path, dirs, files);
        this.clearValue();
      } catch (error) {
        console.error(error);
      }
    }
  },

  deleteItem: async function () {
    if (this.selected.length) {
      if (confirm("Вы действительно хотите удалить?")) {
        try {
          const response = await axios.post(
            `${server}/manager/delete`,
            {
              path: this.path,
              items: this.selected,
            }
          );
          const { path, dirs, files } = response.data;
          this.assignValue(path, dirs, files);
          this.clearValue();
        } catch (error) {
          console.error(error);
        }
      }
    }
  },

  assignValue: function (path: [], dirs: [], files: []) {
    this.path = path;
    this.folders = dirs;
    this.files = files;
  },

  clearValue: function () {
    this.form = "";
    this.select = false;
    this.selected = [];
    this.copied = [];
  },
});

function itemUpdate() {
  fileManager.value.updateItem();
};

function fileType(file: string): string {
  const fileExtension = file.split(".").pop();

  switch (fileExtension) {
    case "txt":
      return "bi-file-text";
    case "json":
      return "bi-file-code";
    case "html":
      return "bi-file-code";
    case "pdf":
      return "bi-file-pdf";
    case "docx":
      return "bi-file-word";
    case "doc":
      return "bi-file-word-fill";
    case "rtf":
      return "bi-earmark-word";
    case "xlsx":
      return "bi-file-excel";
    case "xlsm":
      return "bi-file-excel-fill";
    case "png":
      return "bi-filetype-png";
    case "jpg":
      return "bi-filetype-jpg";
    case "bmp":
      return "bi-filetype-bmp";
    case "zip":
      return "bi-file-zip";
    case "msg":
      return "bi-envelope-paper";
    default:
      return "bi-file";
  }
}
</script>

<template>
  <div class="container py-3">
    <div class="text-opacity-85 text-primary mb-5">
    <h3>Файловый менеджер</h3>
    </div>
    <div class="border border-primary p-5">
      <div class="row border border-primary p-3">
        <div class="col-1">
          <button
            type="button"
            class="btn btn-outline-primary"
            @click="fileManager.getFoldersFiles"
          >
            <i class="bi bi-house" title="Домой"></i>
          </button>
        </div>

        <div class="col-1" :disabled="!fileManager.path.length">
          <button
            type="button"
            class="btn btn-outline-primary"
            title="Родитель"
            @click="
              fileManager.path = fileManager.path.slice(0, -1);
              fileManager.openFolder('parent');
            "
            :disabled="fileManager.select"
          >
            <i class="bi bi-arrow-90deg-up"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            :class="fileManager.action === 'create' ? 'btn btn-primary' : 'btn btn-outline-primary'"
            @click="
              fileManager.action = 'create';
              fileManager.updateItem();
            "
            :disabled="fileManager.select"
          >
            <i class="bi bi-plus-square" title="Создать папку"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            class="btn btn-outline-primary"
            @click="
              fileManager.select = !fileManager.select;
              fileManager.selected = [];
            "
          >
            <i class="bi bi-check-square" title="Выбрать"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            :class="fileManager.action === 'copy' ? 'btn btn-primary' : 'btn btn-outline-primary'"
            @click="
              fileManager.action = 'copy';
              fileManager.copied = fileManager.path;
              fileManager.select = false;
            "
            :disabled="!fileManager.select || fileManager.selected.length === 0"
          >
            <i class="bi bi-clipboard" title="Копировать"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            :class="fileManager.action === 'cut' ? 'btn btn-primary' : 'btn btn-outline-primary'"
            @click="
              fileManager.action = 'сut';
              fileManager.copied = fileManager.path;
              fileManager.select = false;
            "
            :disabled="!fileManager.select || fileManager.selected.length === 0"
          >
            <i class="bi bi-scissors" title="Вырезать"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            class="btn btn-outline-primary"
            :disabled="fileManager.select || fileManager.selected.length === 0"
            @click="fileManager.copyItem"
          >
            <i class="bi bi-clipboard-fill" title="Вставить"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            :class="fileManager.action === 'rename' ? 'btn btn-primary' : 'btn btn-outline-primary'"
            @click="fileManager.action === 'rename' ? fileManager.action = '' : fileManager.action = 'rename'"
            :disabled="!fileManager.select || fileManager.selected.length !== 1"
          >
            <i class="bi bi-pencil" title="Переменовать"></i>
          </button>
        </div>

        <div class="col-1">
          <button
            type="button"
            class="btn btn-outline-primary"
            :disabled="!fileManager.select || fileManager.selected.length === 0"
            @click="fileManager.deleteItem"
          >
            <i class="bi bi-trash" title="Удалить"></i>
          </button>
        </div>
      </div>

      <div class="py-3">
        <nav aria-label="breadcrumb" :disabled="fileManager.select">
          <ol class="breadcrumb">
            <li
              class="breadcrumb-item active"
              aria-current="page"
              v-for="(item, idx) in fileManager.path"
              :key="item"
            >
              <a
                href="#"
                @click="
                  fileManager.path = fileManager.path.slice(0, idx);
                  fileManager.openFolder('open', item);
                "
              >
                {{ item }}
              </a>
            </li>
          </ol>
        </nav>
      </div>

      <div id="fileManager">
        <div
          v-for="(folder, idx) in fileManager.folders"
          :key="idx"
          class="row"
        >
          <div class="item-wrapper fs-6">
            <input
              class="form-check-input"
              type="checkbox"
              v-if="fileManager.select"
              :value="folder"
              v-model="fileManager.selected"
            />
            &nbsp;
            <a
              type="button"
              class="btn btn-link btn-lg text-decoration-none"
              @click="fileManager.openFolder('open', folder)"
              :disabled="fileManager.select"
            >
              <i class="bi bi-folder"></i>
              {{ folder }}
            </a>
            <ManagerForm 
              v-if="fileManager.action === 'rename' && fileManager.selected[0] === folder"
              v-model="fileManager.form"
              @update-item="itemUpdate()"
            />
          </div>
        </div>
      </div>
      <div v-for="(file, idx) in fileManager.files" :key="idx" class="row">
        <div class="item-wrapper fs-6">
          <input
            class="form-check-input"
            type="checkbox"
            v-if="fileManager.select"
            :value="file"
            v-model="fileManager.selected"
          />
          &nbsp;
          <a
            type="button"
            class="btn btn-link btn-lg text-decoration-none"
            @click="fileManager.openFile(file)"
            :disabled="fileManager.select"
          >
            <i :class="fileType(file)"></i>
            {{ file }}
          </a>
          <ManagerForm
            v-if="fileManager.action === 'rename' && fileManager.selected[0] === file"
            v-model="fileManager.form"
            @update-item="itemUpdate()"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
#fileManager {
  height: 75vh;
  overflow-y: auto;
}
.item-wrapper {
  display: flex;
  align-items: center;
}
</style>
