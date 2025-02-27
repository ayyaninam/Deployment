### How to Point Domain and Deploy Django Project using Github on Gunicorn & Nginx Remote Server or VPS
#### What is Gunicorn ?
###### Gunicorn is a Web Server Gateway Interface (WSGI) which receive requests sent to the Web Server from a Client and forwards them onto the Python applications or Web Frameworks e.g. Flask or Django in order to run the appropriate application code for the request. It basically provides a bridge to communicate between your Web Server and Web Application.
#
- On Local Machine, Goto Your Project Folder then follow below instruction:
    - Open Terminal
    - Activate Your virtual Env
    - Install Django Extensions Package It will help to clear pyc and cache (Optional)
    ```sh
      pip install django-extensions
    ```
     - Add Django Extensions Package to INSTALLED_APPS in settings.py File
    ```sh
      INSTALLED_APPS = (
        ...
        'django_extensions',
        ...
      )
    ```
    - Create requirements.txt File
    ```sh
      pip freeze > requirements.txt
    ```
    - Deactivate Virtual Env
- Get Access to Remote Server via SSH
```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
Example:- ssh -p 1034 ayyan@216.32.44.12
```
- Verify that all required softwares are installed
```sh
nginx -v
python --version  OR python3 --version
pip --version
- SQLite is Included with Python
  python -c "import sqlite3; print(sqlite3.sqlite_version)"
git --version
```
- Install Software (If required)
```sh
sudo apt install nginx
sudo apt install python
sudo apt install python3-pip
sudo apt install git
```
- Install virtualenv
```sh
pip list
sudo pip install virtualenv
OR
sudo apt install python3-virtualenv
```
- Verify Nginx is Active and Running
```sh
sudo service nginx status
```
- Verify Web Server Ports are Open and Allowed through Firewall
```sh
sudo ufw status verbose
```
- Exit from Remote Server
```sh
exit
```
- Login to Your Domain Provider Website
- Navigate to Manage DNS
- Add Following Records:

| Type | Host/Name | Value |
| :---: | :---: | :--- |
| A     | @     | Your Remote Server IP |
| A     | www   | Your Remote Server IP |
| AAAA  | @     | Your Remote Server IPv6 |
| AAAA  | www   | Your Remote Server IPv6 |

- Copy Project from Local Machine to Remote Server or VPS. There are two ways to do it:-
  1. Using Command Prompt
      - On Local Windows Machine Make Your Project Folder a Zip File using any of the software e.g. winzip
      - Open Command Prompt
      - Copy Zip File from Local Windows Machine to Linux Remote Server
      ```sh
      Syntax:- scp -P Remote_Server_Port Source_File_Path Destination_Path
      Example:- scp -P 1034 backend.zip ayyan@216.32.44.12:
      ```
      - Copied Successfully
      - Get Access to Remote Server via SSH
      ```sh
      Syntax:- ssh -p PORT USERNAME@HOSTIP
      Example:- ssh -p 1034 ayyan@216.32.44.12
      ```
      - Unzip the Copied Project Zip File
      ```sh
      Syntax:- unzip zip_file_name
      Example:- unzip backend.zip
      ```
      
  2. Using Github
      - Open Project on VS Code then Create a .gitignore File (If needed)
      - Push your Poject to Your Github Account as Private Repo
      - Make Connection between Remote Server and Github Repo via SSH Key
      - Generate SSH Keys
      ```sh
      Syntax:- ssh-keygen -t ed25519 -C "your_email@example.com"
      ```
      - If Permission Denied then Own .ssh then try again to Generate SSH Keys
      ```sh
      Syntax:- sudo chown -R user_name .ssh
      Example:- sudo chown -R ayyan .ssh
      ```
      - Open Public SSH Keys then copy the key
      ```sh
      cat ~/.ssh/id_ed25519.pub
      ```
      - Go to Your Github Repo
      - Click on Settings Tab
      - Click on Deploy Keys option from sidebar
      - Click on Add Deploy Key Button and Paste Remote Server's Copied SSH Public Key then Click on Add Key
      - Clone Project from your github Repo using SSH Path It requires to setup SSH Key on Github
      ```sh
      Syntax:- git clone ssh_repo_path
      Example:- git clone git@github.com:geekyshow1/backend.git
      ```
- Create Virtual env
```sh
cd ~/project_folder_name
Syntax:- virtualenv env_name
Example:- virtualenv mb
```
- Activate Virtual env
```sh
Syntax:- source virtualenv_name/bin/activate
Example:- source mb/bin/activate
```
- Install Dependencies
```sh
pip install -r requirements.txt
```
- Install Gunicorn
```sh
pip install gunicorn
```
- Deactivate Virtualenv
```sh
deactivate
```
- Create System Socket File for Gunicorn
```sh
Syntax:- sudo nano /etc/systemd/system/your_domain.gunicorn.socket
Example:- sudo nano /etc/systemd/system/backend.ai.gunicorn.socket
```
- Write below code inside backend.ai.gunicorn.socket File
```sh
Syntax:- 
[Unit]
Description=your_domain.gunicorn socket

[Socket]
ListenStream=/run/your_domain.gunicorn.sock

[Install]
WantedBy=sockets.target

Example:- 
[Unit]
Description=backend.ai.gunicorn socket

[Socket]
ListenStream=/run/backend.ai.gunicorn.sock

[Install]
WantedBy=sockets.target
```
- Create System Service File for Gunicorn
```sh
Syntax:- sudo nano /etc/systemd/system/your_domain.gunicorn.service
Example:- sudo nano /etc/systemd/system/backend.ai.gunicorn.service
```
- Write below code inside backend.ai.gunicorn.service File
```sh
Syntax:-
[Unit]
Description=your_domain.gunicorn daemon
Requires=your_domain.gunicorn.socket
After=network.target

[Service]
User=username
Group=groupname
WorkingDirectory=/home/username/project_folder_name
ExecStart=/home/username/project_folder_name/virtual_env_name/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/your_domain.gunicorn.sock \
          inner_project_folder_name.wsgi:application

[Install]
WantedBy=multi-user.target

Example:-
[Unit]
Description=backend.ai.gunicorn daemon
Requires=backend.ai.gunicorn.socket
After=network.target

[Service]
User=ayyan
Group=ayyan
WorkingDirectory=/home/ayyan/backend
ExecStart=/home/ayyan/backend/mb/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/backend.ai.gunicorn.sock \
          backend.wsgi:application

[Install]
WantedBy=multi-user.target
```
- Start Gunicorn Socket and Service
```sh
Syntax:- sudo systemctl start your_domain.gunicorn.socket
Example:- sudo systemctl start backend.ai.gunicorn.socket

Syntax:- sudo systemctl start your_domain.gunicorn.service
Example:- sudo systemctl start backend.ai.gunicorn.service
```
- Enable Gunicorn Socket and Service
```sh
Syntax:- sudo systemctl enable your_domain.gunicorn.socket
Example:- sudo systemctl enable backend.ai.gunicorn.socket

Syntax:- sudo systemctl enable your_domain.gunicorn.service
Example:- sudo systemctl enable backend.ai.gunicorn.service
```
- Check Gunicorn Status
```sh
sudo systemctl status backend.ai.gunicorn.socket
sudo systemctl status backend.ai.gunicorn.service
```
- Restart Gunicorn (You may need to restart everytime you make change in your project code)
```sh
sudo systemctl daemon-reload
sudo systemctl restart backend.ai.gunicorn
```
- Create Virtual Host File
```sh
Syntax:- sudo nano /etc/nginx/sites-available/your_domain
Example:- sudo nano /etc/nginx/sites-available/backend.ai
```
- Write following Code in Virtual Host File
```sh
Syntax:-
server{
    listen 80;
    listen [::]:80;
    
    server_name your_domain www.your_domain;
    
    location = /favicon.ico { access_log off; log_not_found off; }
    
    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/run/your_domain.gunicorn.sock;
    }
    
    location  /static/ {
        root /var/www/project_folder_name;
    }

    location  /media/ {
        root /var/www/project_folder_name;
    }
}

Example:-
server{
    listen 80;
    listen [::]:80;

    server_name backend.ai www.backend.ai;

    location = /favicon.ico { access_log off; log_not_found off; }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/run/backend.ai.gunicorn.sock;
    }

    location  /static/ {
        root /var/www/backend;
    }

    location  /media/ {
        root /var/www/backend;
    }
}
```
- Enable Virtual Host or Create Symbolic Link of Virtual Host File
```sh
Syntax:- sudo ln -s /etc/nginx/sites-available/virtual_host_file /etc/nginx/sites-enabled/virtual_host_file
Example:- sudo ln -s /etc/nginx/sites-available/backend.ai /etc/nginx/sites-enabled/backend.ai
```
- Check Configuration is Correct or Not
```sh
sudo nginx -t
```
- Restart Nginx
```sh
sudo service nginx restart
```
- Fix Error:DisallowedHost at / Invalid HTTP_HOST header: 
    - Open Django Project settings.py
    ```sh
    cd ~/project_folder_name/inner_project_folder_name
    nano settings.py
    ```
    - Make below changes
    ```sh
    ALLOWED_HOST = ["your_domain"]
    
    Example:-
    ALLOWED_HOST = ["backend.ai", "www.backend.ai"]
    ```
    - Restart Gunicorn (You need to restart everytime you make change in your project code)
    ```sh
    sudo systemctl daemon-reload
    sudo systemctl restart backend.ai.gunicorn
    ```
- Create required Directories inside /var/www We will use it to serve static and media files only
```sh
cd /var/www
sudo mkdir project_folder_name
cd project_folder_name
sudo mkdir static media
```
- Make User, Owner of /var/www/project_folder_name
```sh
cd /var/www
Syntax:- sudo chown -R user:user project_folder_name
Example:- sudo chown -R ayyan:ayyan backend
```
- If we want to use Development's Media Files then We should move development's media files to public directory (Optional)
```sh
cd ~/project_folder_name
Syntax:- sudo mv media/* /var/www/project_folder_name/media/
Example:- sudo mv media/* /var/www/backend/media/
```
- Open Django Project settings.py
```sh
cd ~/project_folder_name/inner_project_folder_name
nano settings.py
```
- Make below changes
```sh
DEBUG = False

STATIC_URL = 'static/'
STATIC_ROOT = "/var/www/backend/static/"

MEDIA_URL = '/media/'
MEDIA_ROOT = "/var/www/backend/media/"
```
- Restart Gunicorn (You need to restart everytime you make change in your project code)
```sh
sudo systemctl daemon-reload
sudo systemctl restart backend.ai.gunicorn
```
- Activate Virtual Env
```sh
cd ~/project_folder_name
source virtualenv_name/bin/activate
```
- Clear pyc Files and Cache. It requires django-extensions package.
```sh
python manage.py clean_pyc
python manage.py clear_cache
```
- Serve Static Files
```sh
python manage.py collectstatic
```
- Create Database Tables
```sh
python manage.py makemigrations
python manage.py migrate
```
- Create Superuser
```sh
python manage.py createsuperuser
```
- If needed Deactivate Virtual env
```sh
deactivate
```
- Restart Gunicorn (You may need to restart everytime you make change in your project code)
```sh
sudo systemctl daemon-reload
sudo systemctl restart backend.ai.gunicorn
```
- Restart Nginx
```sh
sudo service nginx restart
```
- Now you can make some changes in your project local development VS Code and Pull it on Remote Server (Only if you have used Github)
- Pull the changes from github repo
```sh
git pull
```
- Restart Gunicorn (You may need to restart everytime you make change in your project code)
```sh
sudo systemctl daemon-reload
sudo systemctl restart backend.ai.gunicorn
```

##
### How to Automate Django Deployment using Github Action
- On Your Local Machine, Open Your Project using VS Code or any Editor
- Create A Folder named .scripts inside your root project folder e.g. backend/.scripts
- Inside .scripts folder Create A file with .sh extension e.g. backend/.scripts/deploy.sh
- Write below script inside the created .sh file
```sh
#!/bin/bash
set -e

echo "Deployment started ..."

# Pull the latest version of the app
echo "Copying New changes...."
git pull origin master
echo "New changes copied to server !"

# Activate Virtual Env
#Syntax:- source virtual_env_name/bin/activate
source mb/bin/activate
echo "Virtual env 'mb' Activated !"

echo "Clearing Cache..."
python manage.py clean_pyc
python manage.py clear_cache

echo "Installing Dependencies..."
pip install -r requirements.txt --no-input

echo "Serving Static Files..."
python manage.py collectstatic --noinput

echo "Running Database migration..."
python manage.py makemigrations
python manage.py migrate

# Deactivate Virtual Env
deactivate
echo "Virtual env 'mb' Deactivated !"

echo "Reloading App..."
#kill -HUP `ps -C gunicorn fch -o pid | head -n 1`
ps aux |grep gunicorn |grep inner_project_folder_name | awk '{ print $2 }' |xargs kill -HUP

echo "Deployment Finished !"
```
- Go inside .scripts Folder then Set File Permission for .sh File
```sh
git update-index --add --chmod=+x deploy.sh
```
- Create Directory Path named .github/workflows inside your root project folder e.g. backend/.github/workflows
- Inside workflows folder Create A file with .yml extension e.g. backend/.github/workflows/deploy.yml
- Write below script inside the created .yml file
```sh
name: Deploy

# Trigger the workflow on push and
# pull request events on the master branch
on:
  push:
    branches: ["master"]
  pull_request:
    branches: ["master"]

# Authenticate to the the server via ssh
# and run our deployment script
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          port: ${{ secrets.PORT }}
          key: ${{ secrets.SSHKEY }}
          script: "cd ~/project_folder_name && ./.scripts/deploy.sh"
```
- Go to Your Github Repo Click on Settings
- Click on Secrets and Variables from the Sidebar then choose Actions
- On Secret Tab, Click on New Repository Secret
- Add Four Secrets HOST, PORT, USERNAME and SSHKEY as below
```sh
Name: HOST
Secret: Your_Server_IP
```
```sh
Name: PORT
Secret: Your_Server_PORT
```
```sh
Name: USERNAME
Secret: Your_Server_User_Name
```
- You can get Server User Name by loging into your server via ssh then run below command
```sh
whoami
```
- Generate SSH Key for Github Action by Login into Remote Server then run below Command OR You can use old SSH Key But I am creating New one for Github Action
```sh
Syntax:- ssh-keygen -f key_path -t ed25519 -C "your_email@example.com"
Example:- ssh-keygen -f /home/ayyan/.ssh/gitaction_ed25519 -t ed25519 -C "gitactionautodep"
```
- Open Newly Created Public SSH Keys then copy the key
```sh
cat ~/.ssh/gitaction_ed25519.pub
```
- Open authorized_keys File which is inside .ssh/authroized_keys then paste the copied key in a new line
```sh
cd .ssh
nano authorized_keys
```
- Open Newly Created Private SSH Keys then copy the key, we will use this key to add New Repository Secret On Github Repo
```sh
cat ~/.ssh/gitaction_ed25519
```
```sh
Name: SSHKEY
Secret: Private_SSH_KEY_Generated_On_Server
```
- Commit and Push the change to Your Github Repo
- Pull the changes from github to remote server just once this time
```sh
cd ~/project_folder_name
git pull
```
- Your Deployment should become automate.
- On Local Machine make some changes in Your Project then Commit and Push to Github Repo It will automatically deployed on Live Server
- You can track your action from Github Actions Tab
- If you get any File Permission error in the action then you have to change file permission accordingly.
- All Done



#CELERY
Certainly! To run Celery and Celery Beat as systemd services on Ubuntu with minimal downtime, you can use `systemctl` to manage these services. This approach provides better integration with the system's init system, allows for automatic restarts, logging, and is generally preferred for production environments.

Below are detailed steps to set up Celery and Celery Beat as systemd services for your Django project `backend` located at `/home/ayyan/backend`.

---

### **Prerequisites**

1. **Django Project Setup:**
   - Your Django project (`backend`) is properly configured.
   - Celery is installed and configured in your project.
   - Celery Beat is installed for periodic tasks.
   - You have a virtual environment set up (recommended).

2. **Celery Configuration:**
   - Ensure you have a `celery.py` file in your project directory and that it's properly set up.
   - Ensure your Django apps have tasks defined and registered.

3. **System User:**
   - Create a dedicated system user for running the Celery services if not already existing (in this case, `ayyan`).

---

### **Step 1: Configure Celery in Your Django Project**

Ensure your Django project is configured to work with Celery:

**`backend/celery.py`:**

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# Set default Django settings module for 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'backend.settings')

app = Celery('backend')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()
```

**`backend/__init__.py`:**

```python
from __future__ import absolute_import, unicode_literals
from .celery import app as celery_app

__all__ = ('celery_app',)
```

Ensure your `settings.py` includes the necessary Celery configurations.

---

### **Step 2: Create systemd Service Files**

#### **2.1 Identify Paths**

- **Project Directory:** `/home/ayyan/backend`
- **Virtual Environment (if used):** For example, `/home/ayyan/backend`
- **Python Executable:** The Python interpreter from your virtual environment or system Python.

#### **2.2 Create Celery Service File**

Create a systemd service file for Celery:

```bash
sudo nano /etc/systemd/system/celery.service
```

**Add the following configuration:**

```ini
[Unit]
Type=forking
User=ayyan
Group=www-data
Environment=DJANGO_SETTINGS_MODULE=backend.settings
Environment=PYTHONPATH=/home/ayyan/backend
WorkingDirectory=/home/ayyan/backend
ExecStart=/home/ayyan/backend/env/bin/celery multi start worker \
    --app=backend.celery:app \
    --concurrency=2 \
    --pool=prefork \
    --prefetch-multiplier=1 \
    --max-tasks-per-child=50 \
    --logfile=/var/log/celery/%n.log \
    --pidfile=/var/run/celery/%n.pid
ExecStop=/home/ayyan/backend/env/bin/celery multi stopwait worker \
    --pidfile=/var/run/celery/%n.pid
ExecReload=/home/ayyan/backend/env/bin/celery multi restart worker \
    --pidfile=/var/run/celery/%n.pid
Restart=always
RestartSec=3
StartLimitBurst=5
StartLimitInterval=10
LimitNOFILE=65536


# Resource limits to prevent excessive memory usage
MemoryLimit=1024M           # Limit to 1 GB per worker process
CPUQuota=50%                # Limit to 50% of a single CPU core



[Install]
WantedBy=multi-user.target
```

#### **2.3 Create Celery Beat Service File**

Create a systemd service file for Celery Beat:

```bash
sudo nano /etc/systemd/system/celerybeat.service
```

**Add the following configuration:**

```ini
[Unit]
Description=Celery Beat Service
After=network.target

[Service]
Type=simple
User=ayyan
Group=www-data
Environment=DJANGO_SETTINGS_MODULE=backend.settings
Environment=PYTHONPATH=/home/ayyan/backend
WorkingDirectory=/home/ayyan/backend
ExecStart=/home/ayyan/backend/env/bin/celery -A backend.celery:app beat --loglevel=IN>
Restart=always
RestartSec=3
StartLimitBurst=5
StartLimitInterval=10

# Memory and CPU limits
MemoryLimit=512M            # Restrict memory usage
CPUQuota=20%                # Restrict CPU usage to 20% of a single core

[Install]
WantedBy=multi-user.target
```

#### **2.4 Create Logging Directory**

Create a directory for Celery logs and set proper permissions:

```bash
sudo mkdir /var/log/celery
sudo chown -R ayyan:www-data /var/log/celery
```

#### **2.5 Create PID Directory**

Create a directory for Celery PID files:

```bash
sudo mkdir /var/run/celery
sudo chown -R ayyan:www-data /var/run/celery
```

---

### **Step 3: Configure Systemd Settings for High Availability**

To ensure minimal downtime, configure systemd to automatically restart the services and limit the number of restarts to prevent a crash loop.

#### **3.1 Modify Restart Policies**

In the `[Service]` section of both `celery.service` and `celerybeat.service`, ensure the following:

```ini
Restart=always
RestartSec=3
StartLimitBurst=5
StartLimitInterval=10
```

This configuration will:

- **Restart=always:** Always restart the service when it stops.
- **RestartSec=3:** Wait 3 seconds before restarting.
- **StartLimitBurst=5:** Allow up to 5 restarts...
- **StartLimitInterval=10:** ...within 10 seconds before systemd gives up.

This prevents rapid restarts that can cause system instability.

---

### **Step 4: Enable and Start the Services**

Reload systemd to recognize the new service files:

```bash
sudo systemctl daemon-reload
```

Enable the services to start on boot:

```bash
sudo systemctl enable celery
sudo systemctl enable celerybeat
```

Start the services:

```bash
sudo systemctl start celery
sudo systemctl start celerybeat
```

---

### **Step 5: Verify the Services**

Check the status of the services to ensure they're running properly:

```bash
sudo systemctl status celery
sudo systemctl status celerybeat
```

You should see that both services are `active (running)`.

---

### **Step 6: Configure Logging**

Ensure logs are being written correctly to `/var/log/celery/`. You can monitor the logs using:

```bash
tail -f /var/log/celery/worker.log
tail -f /var/log/celery/beat.log
```

---

### **Additional Considerations for High Availability**

#### **6.1 Optimize Celery Workers**

- **Concurrency:** Adjust the number of worker processes according to your server's CPU cores.

  In your `ExecStart` command for `celery.service`, add the `--concurrency` option:

  ```ini
  ExecStart=/home/ayyan/backend/env/bin/celery multi start worker --app=backend.celery:app --concurrency=4 --logfile=/var/log/celery/worker.log --pidfile=/var/run/celery/%n.pid
  ```

  Replace `4` with the optimal number of processes for your server.

- **Prefetching:** Optimize task prefetching to balance task throughput and latency.

#### **6.2 Use a Process Supervisor**

While systemd handles process management, using Celery's built-in `celery multi` allows for managing multiple worker processes.

#### **6.3 Monitoring**

Implement monitoring to detect and respond to failures promptly:

- **Flower:** A web-based tool for monitoring and administrating Celery clusters.

  Install Flower:

  ```bash
  pip install flower
  ```

  Run Flower as a service similar to Celery and Celery Beat.

#### **6.4 Database and Broker High Availability**

Ensure your message broker (e.g., RabbitMQ, Redis) and database are configured for high availability to prevent them from being single points of failure.

#### **6.5 Periodic Health Checks**

Set up health checks to monitor the status of your Celery workers and Celery Beat scheduler.

---

### **Security Considerations**

- **User Permissions:** Ensure the `User` and `Group` settings in your service files have the necessary permissions to access your project files and virtual environment.

- **Environment Variables:** If your project requires specific environment variables, include them in the `[Service]` section using `Environment=` or load them from an environment file with `EnvironmentFile=`.

---

### **Testing the Setup**

After setting up, perform the following tests:

1. **Restart Services:**

   ```bash
   sudo systemctl restart celery
   sudo systemctl restart celerybeat
   ```

2. **Simulate Failure:**

   - Kill a Celery worker process and observe if systemd restarts it automatically.

3. **Run Tasks:**

   - Queue tasks and ensure they are executed.
   - Schedule periodic tasks and confirm they are running as expected.

---

### **Summary**

By configuring Celery and Celery Beat as systemd services with appropriate restart policies and monitoring, you achieve a robust setup with minimal downtime. This setup ensures:

- Automatic restarts in case of failures.
- Integration with the system's init system.
- Centralized logging.
- Enhanced security through controlled user permissions.

---

### **Final Notes**

- **Documentation:** Keep documentation of your configurations for maintenance purposes.
- **Updates:** Regularly update Celery, Django, and other dependencies to incorporate security patches and improvements.
- **Backups:** Ensure you have backups of your configuration files and data.

---

If you need further assistance or have questions about specific configurations, feel free to ask!
