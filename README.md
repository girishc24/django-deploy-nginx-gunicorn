"## Deploy Django Application using gunicorn & Nginx in Production on AWS Ubuntu server" 
# we will see how to use nginx with gunicorn to serve django applications in production.Django is a very powerful web framework and ships with a server which is able to facilitate development. This development server is not scalable and is not suited for production. Hence we need to configure gunicorn to get better scalability and nginx can be used as a reverse proxy and as a web server to serve static files. Let's get started 
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

