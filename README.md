"# Deploy Django Application using gunicorn & Nginx in Production on AWS Ubuntu server" 

"# we will see how to use nginx with gunicorn to serve django applications in production.Django is a very powerful web framework and ships with a server which is able to facilitate development. This development server is not scalable and is not suited for production. Hence we need to configure gunicorn to get better scalability and nginx can be used as a reverse proxy and as a web server to serve static files. Let's get started" 
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="images/5.webp">
</picture>


# Step 1 - Installing python and nginx
### Let's update the server's package index using the command below:

```
sudo apt update
```

```
sudo apt install python3-pip python3-dev nginx
```
### This will install python, pip and nginx server

# Step 2 - Creating a python virtual environment 

```
sudo pip3 install virtualenv
```
"This will install a virtual environment package in python. Let's create a project directory to host our Django application and create a virtual environment inside that directory."
```
git clone https://github.com/girishc24/django-deploy-nginx-gunicorn.git

cd ~/django-deploy-nginx-gunicorn

```
"then"
```
virtualenv env
```
"A virtual environment named env will be created. Let's activate this virtual environment:"
```
source env/bin/activate
```
## Step 3 - Installing Django and gunicorn
```
pip install django gunicorn

pip install -r requirements.txt
```
"This installs Django and gunicorn in our virtual environment"
## Step 4 - Setting up our Django project
"Add your IP address or domain to the ALLOWED_HOSTS variable in settings.py."
```
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
```
## Add this line, “whitenoise.runserver_nostatic”, into your Installed_apps of setting file.add these lines at the bottom of the blog/urls. py file.
```
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
elif getattr(settings, 'FORCE_SERVE_STATIC', False):
    settings.DEBUG = True
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    settings.DEBUG = False
```
##add these imports lines at the top of the blog/urls. py file
""from django.conf import settings # new
from  django.conf.urls.static import static #new""

## Let's test this sample project by running the following commands:
```
sudo ufw allow 8000
```
## This opens port 8000 by allowing it over the firewall. Let's start our Django development server to test the setup so far:
```
~/projectdir/manage.py runserver 0.0.0.0:8000

```
## Step 5 - Configuring gunicorn
"Lets test gunicorn's ability to serve our application by firing the following commands:"
```
gunicorn --bind 0.0.0.0:8000 textutils.wsgi
```
"This should start gunicorn on port 8000. We can go back to the browser to test our application. Visiting http://<ip-address>:8000 shows a page like this:"

"Deactivate the virtualenvironment by executing the command below:"
```
deactivate
```
"Let's create a system socket file for gunicorn now:"
```
sudo vim /etc/systemd/system/gunicorn.socket
```
"Paste the contents below and save the file"
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```
"Next, we will create a service file for gunicorn"
```
sudo vim /etc/systemd/system/gunicorn.service
```
"Paste the contents below inside this file:"
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=harry
Group=www-data
WorkingDirectory=/home/harry/projectdir
ExecStart=/home/harry/projectdir/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          textutils.wsgi:application

[Install]
WantedBy=multi-user.target
```
"Lets now start and enable the gunicorn socket"
```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```
# Step 6 - Configuring Nginx as a reverse proxy
"Create a configuration file for Nginx using the following command"
```
sudo vim /etc/nginx/sites-available/textutils
```
"Paste the below contents inside the file created"
```
server {
    listen 80;
    server_name www.codewithharry.in;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/harry/projectdir;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```
"Activate the configuration using the following command:"
```
sudo ln -s /etc/nginx/sites-available/textutils /etc/nginx/sites-enabled/
```
"Restart nginx and allow the changes to take place."
```
sudo systemctl restart nginx
sudo service gunicorn restart
sudo service nginx restart
```