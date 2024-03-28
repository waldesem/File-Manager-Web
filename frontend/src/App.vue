<script setup lang="ts">
import { defineAsyncComponent } from "vue";

const FileManager = defineAsyncComponent(
  () => import("./components/FileManager.vue")
);
</script>

<template>
  <FileManager />
</template>
