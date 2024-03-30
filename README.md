# FileManager-Web (in dev now)

FileManager-Web is a web interface for managing a localhost folders and files.

### The main technology stack used in this project includes:

- Golang;
- Fiber;
- Vue3;
- Bootstrap

### Usage

To start the application at http://localhost:3000 run the following command in your terminal:

```
go run main.go

```

### Node Development (optional)

You will also need to have Node.js installed on your machine to build and run the TypeScript code.

```
npm i
```

To start development node server run the following command in your terminal:

```
npm run dev
```

To build the code in the static directory, first comment/uncomment the lines `server` in ./frontend/components/FileManager.vue
Then run:

```
npm run build
```

This will compile the TypeScript code and output the JavaScript and CSS files in the static directory '/backend/app/static'.

### License

This project is licensed under the MIT License.
