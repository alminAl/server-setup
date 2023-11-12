# Django App Deploy


## Creating a non-root user
```bash
adduser <username>
```
- You will be asked to enter some information and a password. Choose a strong password to avoid getting hacked!
- We have successfully created an account with basic access. We want this user to be able to elevate to root access when required. This will be useful when you want to use your server and sometimes do something which requires root access.
- This will save you the time to log out and log back in as the root user.

Execute the following command on the server:
```bash
  usermod -aG sudo <username>
```

## Firewall setup
```bash
  adduser <username>
```
Our server is public, and we want to protect it from external unwanted connections. We want only selected services exposed to the public on our server.

We will set up the UFW firewall on our Ubuntu 20.04 server and allow OpenSSH, which allows us to connect to our server.

Execute the following command to allow OpenSSH:
```bash
ufw allow OpenSSH
```

Enable the firewall by executing the following command:
```bash
ufw enable
```

You can type the following command to elevate to the root access as and when required.
```bash
sudo su
```

```bash
exit
```

## Git Clone
```bash
cd ~/.ssh
```
```bash
ssh-keygen -t ecdsa -b 521 -C "root@<server username>"
```
```bash
touch ~/.ssh/config
```
```bash
chmod 0600 ~/.ssh/config
```
```bash
chown root:root~/.ssh/config
```

```bash
Host <ApiBackend>
    HostName github.com
    IdentityFile ~/.ssh/<any_file_name>
```

```bash
ssh -T git@<ApiBackend>.com
```
clone project /home/<username>
```bash
git@<ApiBackend>:theyolostudio/ApiBackend.git
```


## Creating a python virtual environment
```bash
sudo pip3 install virtualenv
```
```bash
virtualenv venv
```
```bash
source venv/bin/activate
```

## Installing Django and gunicorn
```bash
pip install gunicorn
```
```bash
sudo ufw allow 8000
```
```bash
gunicorn --bind 0.0.0.0:8000 ApiBackend.wsgi
```
```bash
deactivate
```

## Creating systemd Socket and Service Files for Gunicorn

Navigate to /etc/systemd/system/

Create a file named: gunicorn.socket

Add the following to the file and save:

```bash
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

## Create gunicorn service to run the WSGI application (the django app)

create new file: gunicorn.service

Add the following to gunicorn.service and save.

It's very important to copy this exactly as I have. Also your directory structure inside /home/django/ must be EXACTLY the same as mine. Otherwise this service file won't know what project you're talking about.


```bash
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=bpm
Group=www-data
WorkingDirectory=/home/bpm/BappaMazumderBackend
ExecStart=/home/bpm/BappaMazumderBackend/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          BappaMazumderBackend.wsgi:application

[Install]
WantedBy=multi-user.target
```

sudo systemctl start gunicorn.socket

sudo systemctl enable gunicorn.socket

### Helpful Commands
1. sudo systemctl daemon-reload
  - Must be executed if you change the gunicorn.service file.
2. sudo systemctl restart gunicorn
- If you change code on your server you must execute this to see the changes take place.
3. sudo systemctl status gunicorn
4. sudo shutdown -r now
- restart the server

## Configure Nginx to Proxy Pass to Gunicorn
```bash
sudo apt install nginx
```
Navigate to /etc/nginx/sites-available

Create file  < Any File Name >

```bash
server {
    server_name <your_ip_address>;

    location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
        alias /home/bpm/BappaMazumderBackend/static/;
    }
	
	location /media/ {
		alias /home/bpm/BappaMazumderBackend/media/;
	}
    
     location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
    

}
```

### Configure the Firewall
sudo ln -s /etc/nginx/sites-available/BappaMazumderBackend  /etc/nginx/sites-enabled

sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 8000

sudo ufw allow 'Nginx Full'

sudo systemctl restart gunicorn

Set DEBUG=False in settings.ini 

if it isn't already.
service gunicorn restart (There is no difference between this command and sudo systemctl restart gunicorn)
Restart the 

server sudo shutdown -r now


Visit http://<your_ip_address>/

## Adiotional Permission
```bash
sudo usermod -a -G bpm www-data
```
```bash
sudo chmod -R 777 media
```










