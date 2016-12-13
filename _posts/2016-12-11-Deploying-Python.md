---
layout: post
tags: post
title: Deploying a Python application on Linode (Flask, Gunicorn, Nginx)
---

Deploying a Python application nowadays is very easy, especially if you know what you're doing. But, if you don't, and you miss just a small part of the process then it is likely that typing your domain in the browser is going to return a 404 not found. This article can be a reference for those that are looking to setup a similar application.


#### Getting a domain
This part is probably the easiest step in the process, there are many vendors you can get the website from. I would recommend getting your domain from a popular vendor because if you run into problems then it is likely you can find the answer to your problem online. Once you get your first website up and running, then you try to experiment with different vendors, services, frameworks, etc.

#### Getting a VM
The next step is to choose a place to host your application, there are many options: AWS, Google, DigitalOcean, Linode, etc. There are different benefits to each, so I would do a little research before choosing. Ultimately all of these companies are going to provide a similar service, so don't sweat it unless you have specific needs.

#### Nameservers and DNS
This step is really important, and kind of difficult to understand if you haven't done it before. When you type an address in your browser, it will go to the nameserver that it originated from (Google, Namecheap, GoDaddy, etc.). And you need to have that nameserver point to the right place so your application can serve the site. This process is different for every name registrar, so you might need to do a little Googling. As an example, using Namecheap to get a domain and Linode a VM. You would have to select `Custom DNS` and then have 5 entries

```
ns1.linode.com
ns2.linode.com
ns3.linode.com
ns4.linode.com
ns5.linode.com
```
This tells Namecheap that you want that domain to be registered with these boxes. Then on Linode's Manager you would have a "domain zone" (your domain) that points to your VM's IP address (usually this is all setup for you, you might just have to select a few things from a dropdown). Every domain provider and hosting company has their own intricacies, so it's probable you will have to read some of their docs to figure out what their process entails for this step.

#### Securing your server
As a first step when you log onto your VM, I would highly recommend securing your server because you never know what could happen, especially when you start publishing your website. I recommend one of [Linode's articles](https://www.linode.com/docs/security/securing-your-server). It is a beginner friendly guide that has explanations for a variety of operating systems.

#### Serving your domain on your box
You will need to add an entries to your `/etc/hosts` file so that when a request hits your box looking for your domain, that you can serve it. Typically `localhost` is already filled out for you, but if not go ahead and add it. You can figure out your hostname by typing `hostname` in the terminal.

```
127.0.0.1       localhost
<VM-IP-addres>  <domain>        <your-hostname>
```


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
The `proxy_pass` field allows Nginx to reverse proxy your site from `localhost:8000` out to the world!
Once you have that setup, then you need to delete the default sites-enabled, and link your new file.

    sudo rm /etc/nginx/sites-enabled/default
    sudo ln -s /etc/nginx/sites-available/<application-name> /etc/nginx/sites-enabled/

And finally

    sudo /etc/init.d/nginx restart

#### Starting up your application

Assuming you have a Flask application that runs (if you don't look at [this quickstart guide](http://flask.pocoo.org/docs/0.11/quickstart/)) then all we need to do is setup the WSGI. If you don't know what a WSGI is, read [about it here](https://www.fullstackpython.com/wsgi-servers.html). We'll use Gunicorn, for Ubuntu you can install with

    sudo apt-get install gunicorn

Then create a file `wsgi.py`. If your application has a `main.py` with a Flask application called `application`, then `wsgi.py` would contain

```
from main import app

if __name__ == "__main__":
    app.run()  
```

To run,

    gunicorn --bind 0.0.0.0:8000 wsgi &

You should now be able to see your website when you type your domain in your browser (assuming you've given enough time for the DNS registrar to register you). Some other steps you may want to look into are

* Startup scripts for your service when the VM boots
* More security
* Scaling across multiple boxes

Hope this helped!
