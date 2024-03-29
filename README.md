# FileManager-Web (in dev now)

FileManager-Web is a web interface for managing a localhost folders and files.

### The main technology stack used in this project includes:

- Python 3;
- Flask;
- Vue3;
- Bootstrap

### Installation

For installiing the required Python packages run the following command in the terminal:

```
sudo apt install python3 python3-pip python3-venv # if not install venv yet
git clone https://github.com/waldesem/File-Manager-Web.git
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Usage

To start the application at http://localhost:5000 run the following command in your terminal:

```
gunicorn -c gunicorn.conf.py wsgi:app  # start the gunicorn server with the settings in gunicorn.conf.py
```

### Node Development (optional)

You will also need to have Node.js installed on your machine to build and run the TypeScript code.
After installing Node.js, you can install the required npm packages by running in your webapp directory the following command in your terminal:

```
npm i
```

To start development node server run the following command in your terminal:

```
npm run dev
```

To build the code in the static directory flask app, first comment/uncomment the lines `server` in /Web-Personal-DB/frontend/src/utilities/utils.ts
Then run the following command in your terminal:

```
npm run build
```

This will compile the TypeScript code and output the JavaScript and CSS files in the static directory '/backend/app/static'.

### License

This project is licensed under the MIT License.
