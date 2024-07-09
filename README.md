
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
all available network interface or IPs
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
![image](https://github.com/ougabriel/ougabriel-devops-stage-2/assets/34310658/4a3f52c5-4323-44f9-ad71-65087b41f0cc)

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


Lets re-login into the app









