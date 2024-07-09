
##STAGE ONE: THE BACKEND

**1. CLONE THE REPO**
In the first stage I cloned the repo directly on my Azure VM
```bash
git clone https://github.com/hngprojects/devops-stage-2
```
When its cloned `cd` into it and do and `ls` to see the content of the repo

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/10fc2d91-1d2d-4c82-b59e-d52835d4ac3a)

**2. CONFIGURE THE BACKEND FIRST**
```bash
cd backend
ls 
cd app
```
When you `ls` the content of the `backend` service we can see the `poetry.lock` file 
This Poetry helps you declare, manage and install dependencies of Python projects, ensuring you have the right stack everywhere.
When you  `cd` into the `app` we can see that this application uses the `sqlachelmy` database engine which is a library that facilitates the communication between the `python` programmes and databases. 
In other words, our database is running on a `python` programme and will be using `postgres` database as given in the task doc.
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/ff82a575-38a8-4fab-bd94-3f8f9ffd84a8)

Based on this new information, we will need to install the `postgres` and `poetry`

      **2a Install Poetry and its dependencies**

```bash
curl -sSL https://install.python-poetry.org | python3 -
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/0bd09c53-1cbf-4399-91b9-cb8baa7eb126)

If you run `poetry --version` and it is still not available as in the image below
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/718e908d-951f-4c8b-bd12-ad3b240efae9)
To resolve this add `poetry ` to your system path. This is the official path
```bash
export PATH="$HOME/.poetry/bin:$PATH" >> ~/.bashrc
source ~./bashrc
poetry --version
```
or use this, depending on where it is installed on your system
```bash
export PATH="$HOME/.local/bin:$PATH" >> ~/.bashrc
source ~./bashrc
poetry --version
```
When you do this,  you should now be able to see the `poetry version`
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/6d1c4f34-2969-4dfa-91ee-c0ab23b9e27d)

Install the poetry dependencies
```bash
poetry install
```
If for any reason like in my case you have this `error` output. You will need to install the `python version` mentioned
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/03189db2-be7d-4b3b-bc36-12e40bd5a118)

Update and install Python
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-dev -y
```
Confirm the python version `python3.10 --version`
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/e35e0856-e986-4af4-b38c-9a44adab1fb9)

Confifure `poetry` to use the new python version
```bash
poetry env use python3.10
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/c79998d5-e564-464a-adf8-7e6176e31224)

When this is done, we can now re-run `poetry install` again


      **2b Install Postgres**
The next action is to configure the database and its permissions.
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```
To use Postgresql and create a user run the following command
```bash
sudo -i -u postgres
psql
```
Create a user  `gabApp` with password `gabApp#Admin`
```sql
CREATE USER app WITH PASSWORD 'my_password';
```
Create a database named `app` and grant all privileges to the `app` user:
```sql
CREATE DATABASE app;
\c app
GRANT ALL PRIVILEGES ON DATABASE app TO app;
GRANT ALL PRIVILEGES ON SCHEMA public TO app;
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/06622ae8-fb20-4a41-bd4e-96d2f8bd3ec9)

Confirm if the USER and DATABASE exist

```sql
\l
\du
```
We can see the newly created  `app` database and the `app` user below. 
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/d4dc4f1a-ebcf-40d1-985f-887331a2a96f)

Exit the PostgreSQL shell and switch back to your regular user.
```sql
\q
exit
```
    **2c. Configure the .env Backend credentials to match with the database
we have to make sure the `.env` variables matches the database credentials created earlier
To see the `.env` file, `cd` into the backend directory if you are not already there and  `ls -al` to see all hidden files and folders
```bash
ls -al
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/59cf58ac-86b3-47a8-83ce-eb24283a5690)

Using `vim` or `nano` change the content to match the database
```sql
vi .env
```
Before:
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/fbc8cae2-f4c9-43e1-b23b-ff2ecb90bfb9)

After:
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/576bfd73-f289-4024-91f5-10bb26a655fc)


Set up the database with the necessary tables:
```bash
poetry run bash ./prestart.sh
```
You should get this output after running the `./prestart.sh` script

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/ca03f7b5-0a0d-4e34-b14c-f25d9309d416)

Next, we have to make the backend server to be open to all traffic connections, which is `0.0.0.0`. This will enable the backend to become accessible on 
all available network interface or IPs. The backend will listen for traffic coming from the frontend on port `8000`
```bash
poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
When you run this command, and you see the 'application startup complete' output. Then all is fine and well at this stage
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/50dc9c80-7ff4-42e0-bc45-a0a1c5546d45)

Press `Ctrl C` to cancel and shutdown the backend application
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/3eb524a7-5bfc-460f-9333-9d07eae51b1c)

###STAGE TWO: THE FRONTEND SERVICE
To configure the frontend service you must exit the backend service.
Move into the frontend service directory
```bash
cd frontend
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/9143562b-250a-4f06-8b3b-74f37615ac52)

When you do an `ls -al` to see all of its contents both the hidden ones, a glance into the file content we can tell the app is built
on nodeJs and NPM dependencies. 

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/b474927a-7ff4-4af7-b22e-6ed5c4961c2d)

**2.1 Install the required packages**
```bash
sudo apt update
sudo apt install nodejs npm
```
After the installation, checked the `npm and node` version to see it is old.
the version installed was low, I need to upgrade my `npm` and `nodejs` versions to a more stable and recent version.
Node version from ^18 should be good.

------------------
NOTE: this step is optional dependent on your node version.
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
Aftwe installing `nvm`, we need it to load it into our current shell session
```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```
Verify `npm` 

```bash
command -v nvm
```
Install the lastest version of nodejs

```bash
nvm install node
```
Verify the version
```bash
npm -v
node -v
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/78325ed8-b6d3-49d3-b99b-9d871735edfd)

---------------------

**2.2 Install Frontend Dependencies and Run the server**
```bash
npm install
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/190eb6e7-d7c0-49ce-9df0-a9d91039f583)

Run the server on all network interface
```bash
npm run dev -- --host
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/c89a934d-52ea-4b00-8fc6-ddcd4dc5104c)

**2.3 Accessing the App and open inbound port 5173 on Azure VM**
We can acccess the App locally by running `curl http://localhost:5173` 
We can access the App UI by running typing on a browser `http://<remote-vm-ip>:5173`

In my case I will be using `http://20.0.113.13:5173`
Before accessing the application we need to open `port 5173` 
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/e02ca381-2936-48dd-a1c6-5f22adcac1c1)

Access the App from the opened port

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/0d4a49f6-183e-48cd-890f-754f9d260171)
Login details are obtained in the backend dir of the `.env` file which are username = `devops@hng.tech` and password = `devops#HNG11`

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/15e17f70-5168-4e2d-8584-14709aa0cf2c)


**2.4 Resolving the Network Error Issue**
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/e138c242-7e7f-4820-ae7e-6dab73efea79)

This error means there is an issue connecting to the backend service of our application from the frontend
When we right click on the page and select `inspect` we can see that at the console section of the developer tools `http://localhost:8000 was refused`
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/c2ec80a2-6124-41cb-b426-2883bb7206e4)
This is because the frontend is still trying to connect to the backend database through the `localhost` address which is invalid because the app is now running on a remote server.
This needs to be changed to the correct IP of the remote server or VM. 
To change this; go into the `backend` dir > `ls -al` to see hidden folders > `vim` into the `.env` dir and make the changes
 ```bash
cd frontend
ls -al
vi .env
```
Before 

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/a7e25392-1b00-4f86-b41b-e8bf9e73aa4f)


After
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/8c113039-4dab-42ea-81f0-a08d5ca3f01d)


**2.5 Resolving the CORS Error issue**
CORS stands for Cross-origin resource sharing.

When we try re-login we will see a new issue, this is because the backend is not properly routed to match our remote server IP
To do this, go into the backend folder > `ls -al` to show all content > vim into `.env` and change the details
```bash
cd backend
ls -al
vi .env
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/4af655a0-2316-4bd7-9e40-3c6ca6cd8a0f)
Changing the file
Before
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/ec47b80a-fc75-4b2d-99ce-c9b4ea058c56)

After
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/76f48ab5-d53b-46d1-90e1-dbf49993303f)


Lets re-login into the app; from the image below, we now have a complete succesful login

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/5a56a7df-5d26-4957-90fd-e9ada5f39fee)

App is succefully deployed locally

**2.6 Access the `docs` and `redoc` documentation path**
You can access the page using `http://<remote-server-ip/docs` and `http://<remote-server-ip/redoc` respectively.
the `docs` landing page
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/1d8c99b1-a3a8-42d1-8a7c-31f68247e321)

the `redoc` landing page
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/9add38d2-d30e-4d31-a766-8b998aee051a)



###STAGE 3: DEPLOY APP TO CONTAINER USING DOCKER-COMPOSE

**3.1 Create `Dockerfile` for frontend and backend**

We will write two `Dockerfile` for both the frontend and backend
A. the frontend `Dockerfile` > go into the directory and create the file
```bash
cd frontend
vi Dockerfile
```
this is the file frontend `Dockerfile` content
```bash
# Use the latest official Node.js image as a base
FROM node:latest

# Set the working directory
WORKDIR /app

# Copy the application files
COPY . .

# Install dependencies
RUN npm install

# Expose the port the development server runs on
EXPOSE 5173

# Run the development server
CMD ["npm", "run", "dev", "--", "--host"]
```

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/64162a68-353f-4636-a946-7f7e6a72ba13)


B. the backend `Dockerfile` > go into the directory and create the file

```bash
cd backend
vi Dockerfile
```
this is the backend `Dockerfile` content
```bash
# Use the latest official Python image as a base
FROM python:latest

# Install Node.js and npm
RUN apt-get update && apt-get install -y \
    nodejs \
    npm

# Install Poetry using pip
RUN pip install poetry

# Set the working directory
WORKDIR /app

# Copy the application files
COPY . .

# Install dependencies using Poetry
RUN poetry install

# Expose the port FastAPI runs on
EXPOSE 8000

# Run the prestart script and start the server
CMD ["sh", "-c", "poetry run bash ./prestart.sh && poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload"]
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/8428f22a-8160-408a-87cd-f50baa28bc2d)


**3.2 Create a `docker-compose.yml` in the root directory**
Go to the root of the repo dir
```bash
cd devops-stage-2
vi docker-compose.yml
```
The `docker-compose.yml` content

```bash
version: '3.8'

services:
  backend:
    build:
      context: ./backend
    container_name: fastapi_app
    ports:
      - "8000:8000"
    depends_on:
      - db
    env_file:
      - ./backend/.env

  frontend:
    build:
      context: ./frontend
    container_name: nodejs_app
    ports:
      - "5173:5173"
    env_file:
      - ./frontend/.env

  db:
    image: postgres:latest
    container_name: postgres_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    env_file:
      - ./backend/.env

  adminer:
    image: adminer
    container_name: adminer
    ports:
      - "8080:8080"

  proxy:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx_proxy_manager
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db
      - backend
      - frontend
      - adminer

volumes:
  postgres_data:
  data:
  letsencrypt:
```

###STAGE 4: MAPPING THE DOMAIN

We need to configure domains and subdomains for our frontend, adminer service, and Nginx proxy manager. Ensure that port 80 directs traffic to both the frontend and backend:

- The main website will be hosted on your domain.
- The backend API will be reachable at yourdomain/api.
- Adminer, used for managing databases, will be found at db.yourdomain.
- Nginx proxy manager will be accessible via proxy.yourdomain.

If you don't have a domain name, you can obtain a subdomain from AfraidDNS, where we acquired our domain for this project. Ensure that each domain and subdomain directs traffic to the server hosting your application.

###STAGE 5: Installing `docker` , `docker-compose` and `nginx-proxy-manager`

A. Install the following required packages
```bash
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
B. more commands to install `docker` by calling and adding the official GPG key and the APT sources
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```
C. Installing docker
```bash
sudo apt-get update
sudo apt-get install docker-ce
sudo groupadd docker
sudo usermod -aG docker $USER
```
D. Start the `docker` and check its status
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/08548c09-aeb5-4d61-b1c6-071d2ae1a735)

E. Installing `docker-compose`
```bash
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")'
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/858c3646-4fac-4e60-a990-48db9926964c)

F. Start the Application
We need to be sure we are in the project `root` directory
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/9ad47cfe-6620-40d7-89ec-51a112527c34)

Run the `docker-compose` file
```bash
docker-compose up -d
```
When we `curl localhost` we can see that `nginx proxy manager` is successfully installed

###STAGE 6: REVERSE PROXY AND SSL CONFIGURATION

**6.1 Access the `nginx proxy manager`**
Access the Proxy manager UI by entering `http://<remote-server-ip>:81` in your browser, Ensure that `port:81` is open in your security group or firewall.
In my Azure VM, I made sure `port:81` is open to all network

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/1b7301e5-75e7-4e7b-ac43-07e6bcce5296)

On the browser `http://20.0.113.13:81`

![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/c169facb-458e-45a0-a6e0-86af5096b9a6)

```bash
#Login details
Email: admin@example.com
Password: changeme
```
**6.2 Setup Proxy for backend and frontend**
Click on Proxy host and setup the proxy for your frontend and backend
Map your domain name to the service name of your frontend and the port the container is listening on Internally.



