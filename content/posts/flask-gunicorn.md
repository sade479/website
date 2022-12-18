---
title: "Deploying a Flask application with Gunicorn"
date: 2022-12-17T15:43:59-05:00
draft: false
---

You have your fancy Flask application working but now you need to deploy it and you have no idea what to do. No worries I was in this same situation in my senior software development project. In this tutorial, I will show you how to deploy your Flask application on a Linux cloud server using Gunicorn.

<!--more-->

## Pre-requisites
1. Cloud Server with SSH access running Ubuntu 22.04 or newer (you can use services such as DigitalOcean, Linode, etc.)
2. Flask application to deploy

## Getting Started

1. Update repositories and install Python
```
sudo apt update
sudo apt install python3-dev python3-pip python3-venv
```
2. Create a directory for your Flask project
```
mkdir ~/project
```
Note you can replace "project" with whatever name you want

3. Create a virtual environment to store Flask and other Python dependencies
```
cd ~/project
python3 -m venv projectenv
```
4. Activate the virtual environment
```
source projectenv/bin/activate
```

Once you have activated your virtual environment your terminal prompt should look something like this:
```
(projectenv) ubuntu@flask-test:~/project$
```
Your environment name should be in parentheses at the start of your terminal prompt. This indicates that you are inside your virtual environment.

## Setting up a Flask application
While still in your virtual environment run the following commands to install all of your pip dependencies.
```
pip install wheel
pip install gunicorn flask
pip install "any other dependencies your specific application needs"
```
Once you have all of your pip dependencies installed you will want to transfer your project files to your server. 

If you are using GitHub to store your application you can simply run ``git clone "your github url here"``.

Otherwise, you will need to use SFTP to transfer your project files to your server. This can be accomplished either using {{<underline "http://winscp.net" WinScp>}} for Windows or {{<underline "http://cyberduck.io" Cyberduck>}} for macOS.

In this tutorial, I will just be using a basic hello world application.
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1>Hello World</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

With your application uploaded to the server, you will first want to test it to make sure everything is working correctly.
You can run it using ``python "yourappname".py``. Once you have done that you can open a web browser and type in your server ip with the corresponding port such as ``10.70.30.68:5000``.

![flask hello world running in firefox](/images/flask-helloworld.png)

You can use CTRL + C to exit the app.

## Setting up Gunicorn

With your Flask application working it is now time to deploy it on an actual application server instead of relying on the built-in development server that Flask uses.

1. Create a new file to serve as your application's entry point
```
touch wsgi.py
```
2. Insert the following code in the newly created file using nano or your preferred text editor
```
from hello import app

if __name__ == "__main__":
    app.run()
```
Replace hello with the name of your app

3. Test Gunicorn by running the following command
```
gunicorn --bind 0.0.0.0:5000 wsgi:app
```
The terminal output should look something like this if everything is working
```
[2022-12-18 18:07:22 +0000] [3591] [INFO] Starting gunicorn 20.1.0
[2022-12-18 18:07:22 +0000] [3591] [INFO] Listening at: http://0.0.0.0:5000 (3591)
[2022-12-18 18:07:22 +0000] [3591] [INFO] Using worker: sync
[2022-12-18 18:07:22 +0000] [3592] [INFO] Booting worker with pid: 3592
```
You can use CTRL + C to exit the app.

4. Deactivate the virtual environment
```
deactivate
```
5. Create a new systemd service for your application
```
sudo nano /etc/systemd/system/hello.service

[Unit]
Description=Gunicorn application server
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/project
Environment="PATH=/home/ubuntu/project/projectenv/bin"
ExecStart=/home/ubuntu/project/projectenv/bin/gunicorn --workers 3 --bind 127.0.0.1:5000 -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```
Note adjust the paths and username to fit your specific needs.
Depending on how many CPU cores your server has you can adjust the workers amount.

6. Enable and start your service
```
sudo systemctl enable hello
sudo service hello start
sudo service hello status
```
The output of the last command should look something like this.
```
 hello.service - Gunicorn application server
     Loaded: loaded (/etc/systemd/system/hello.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-12-18 18:17:44 UTC; 42s ago
   Main PID: 3668 (gunicorn)
      Tasks: 4 (limit: 4569)
     Memory: 55.4M
        CPU: 371ms
     CGroup: /system.slice/hello.service
             ├─3668 /home/ubuntu/project/projectenv/bin/python3 /home/ubuntu/project/projectenv/bin/g>
             ├─3669 /home/ubuntu/project/projectenv/bin/python3 /home/ubuntu/project/projectenv/bin/g>
             ├─3670 /home/ubuntu/project/projectenv/bin/python3 /home/ubuntu/project/projectenv/bin/g>
             └─3671 /home/ubuntu/project/projectenv/bin/python3 /home/ubuntu/project/projectenv/bin/g>

Dec 18 18:17:44 flask-test systemd[1]: Started Gunicorn application server.
Dec 18 18:17:44 flask-test gunicorn[3668]: [2022-12-18 18:17:44 +0000] [3668] [INFO] Starting gunicor>
Dec 18 18:17:44 flask-test gunicorn[3668]: [2022-12-18 18:17:44 +0000] [3668] [INFO] Listening at: un>
Dec 18 18:17:44 flask-test gunicorn[3668]: [2022-12-18 18:17:44 +0000] [3668] [INFO] Using worker: sy>
Dec 18 18:17:44 flask-test gunicorn[3669]: [2022-12-18 18:17:44 +0000] [3669] [INFO] Booting worker w>
Dec 18 18:17:44 flask-test gunicorn[3670]: [2022-12-18 18:17:44 +0000] [3670] [INFO] Booting worker w>
Dec 18 18:17:44 flask-test gunicorn[3671]: [2022-12-18 18:17:44 +0000] [3671] [INFO] Booting worker w>
```

## Using Caddy to route requests

With your application successfully up and running using Gunicorn, you will now want to install a web server to route requests to your application.

I recommenced using Caddy. I have a basic tutorial linked {{<underline "/hosting-a-website-with-caddy" here>}}.

Edit your CaddyFile with the reverse proxy directive it should look something like this.
```
:80 {
        reverse_proxy 127.0.0.1:5000
}
```
Note ``:80`` can be replaced with your domain name for automatic HTTPS support

At this point, you should be able to type in the domain name or ip address of your server directly into your web browser without a port and be able to visit your application.

![flask hello world running in firefox](/images/flask-caddy.png)

## Conclusion

Congratulations your Flask application is now up and running on your server. I hope this tutorial has helped you.