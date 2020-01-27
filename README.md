# Setup your app behind NGINX reverse proxy with Let's Encrypt

This guide is for people using ubuntu server.

#### Plan of action

1. Update server
1. Install nginx
2. Setup nginx configurations
3. Verify nginx configurations
4. Reload nginx service
5. Let's Encrypt for HTTPS

#### Update server

    sudo apt update

#### Install nginx

    sudo apt install nginx

Nginx might not start automatically

    sudo /etc/init.d/nginx start
    
    or

    sudo service nginx start

To check if service is running

    sudo /etc/init.d/nginx status
    
    or
    
    sudo service nginx status

#### Setup nginx configurations

Go to config folder of sites in nginx

    cd /etc/nginx/sites-enabled

Create your site configuration

    sudo vi <git.example.com>.conf

Paste the below code there. Modify `server_name` and `upstream` uri according to your need.

    server {
        server_name <git.example.com>;
        # The internal IP of the VM that hosts your app
        set $upstream <0.0.0.0:4200>;
        location / {
            proxy_pass_header Authorization;
            proxy_pass http://$upstream;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering off;
            client_max_body_size 0;
            proxy_read_timeout 36000s;
            proxy_redirect off;
        }
        listen 80;
    }

#### Verify nginx configurations

To verify whether your configurations are current, run the command below.

    sudo nginx -t

#### Reload nginx service

You're going to have to reload the nginx service for the changes we just made.

    sudo service nginx reload

#### Let's Encrypt for HTTPS

Certbot packages provided by Ubuntu tend to be outdated. However, the Certbot developers maintain a Ubuntu software repository with up-to-date versions, so we’ll use that repository instead.
Let us install all the dependencies first to proceed.

To add repository you need `software-properties-common`

    sudo apt install software-properties-common

Add repository

    sudo add-apt-repository ppa:certbot/certbot
    
You’ll need to press ENTER to accept.

Install Certbot’s Nginx package

    sudo apt install python-certbot-nginx

We now need to get the http changed to https. Run the command below, certbot will automatically find the sites running via nginx.

    sudo certbot --nginx
    
    or
    
    sudo certbot --nginx -d git1.example.com -d git2.example.com

This runs certbot with the `--nginx` plugin, you can also use `-d` to specify the names in one liner.

Follow the on-screen instructions to auto SSL configs.
