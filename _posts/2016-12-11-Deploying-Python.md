---
layout: post
tags: post
title: Deploying a Python application on Linode (Flask, Gunicorn, Nginx)
---

Deploying a Python application nowadays is very easy, especially if you know what you're doing. But, if you don't, and you miss just a small part of the process then it is likely that typing your domain in the browser is going to return a 404 not found. This article can be a reference for those that are looking to setup a similar application.


#### Getting a domain
This part is probably the easiest step in the process, there are many vendors you can get the website from. I would recommend getting your domain from a popular vendor because if you run into problems then it is likely you can find the answer to your problem online. Once you get your first website up and running, then you try to experiment with different vendors, services, frameworks, etc.

#### Getting a VM
The next step is to choose a place to host your application, there are many options: AWS, Google, DigitalOcean, Linode, etc. For my first application I chose Linode because I had some credits and it was relatively easy to setup. There are different benefits to each, so I would do a little research before choosing. Ultimately all of these companies are going to provide a similar service, so don't sweat it unless you have specific needs.

#### Securing your server
I would highly recommend securing your server because you never know what could happen, especially when you start publishing your website. I recommend one of [Linode's articles](https://www.linode.com/docs/security/securing-your-server). It is a beginner friendly guide that has explanations for a variety of operating systems.

#### Configuring Nginx
If you haven't already, install Nginx on the box, if you're using Ubuntu
    sudo apt-get install nginx

Then create a file named after your application in
    touch /etc/nginx/sites-available/<application-name>
Add the following lines to the file, and remember to fill in your username and the app folder's name:

```
server {
    location / {
            proxy_pass http://localhost:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
    }
    location /static {
            alias /home/<user>/<app-folder>/static/;
    }
}
```
The `proxy_pass` field allows Nginx to reverse proxy your site from `localhost:8000` out to
Once you have that setup, then you need to delete the default sites-enabled, and link your new file.
    sudo rm /etc/nginx/sites-enabled/default
    sudo ln -s /etc/nginx/sites-available/<application-name> /etc/nginx/sites-enabled/

And finally
    sudo /etc/init.d/nginx restart

#### Starting up your application

Assuming you have a Flask application that runs (if you don't look at [this quickstart guide](http://flask.pocoo.org/docs/0.11/quickstart/)) then 

gunicorn --bind 0.0.0.0:8000 wsgi
