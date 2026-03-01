# Setup Your App Behind NGINX Reverse Proxy with Let's Encrypt

This guide is for people using Ubuntu server (20.04+).

#### Plan of action

1. Update server
2. Install nginx
3. Setup nginx configurations
4. Verify nginx configurations
5. Reload nginx service
6. Let's Encrypt for HTTPS
7. Verify auto-renewal

#### Update server

    sudo apt update && sudo apt upgrade -y

#### Install nginx

    sudo apt install nginx

Start and enable nginx to run on boot:

    sudo systemctl start nginx
    sudo systemctl enable nginx

To check if the service is running:

    sudo systemctl status nginx

#### Setup nginx configurations

Create your site configuration in `sites-available` and symlink it to `sites-enabled`:

    sudo vi /etc/nginx/sites-available/git.example.com.conf

Paste the below code there. Modify `server_name` and `upstream` uri according to your need.

    server {
        server_name git.example.com;

        # The internal IP of the VM that hosts your app
        set $upstream 127.0.0.1:4200;

        location / {
            proxy_pass http://$upstream;
            proxy_pass_header Authorization;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering off;
            client_max_body_size 50m;
            proxy_read_timeout 300s;
            proxy_redirect off;
        }

        # Security headers [1]
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;
        add_header X-XSS-Protection "1; mode=block" always;

        listen 80;
    }

Enable the site by creating a symlink:

    sudo ln -s /etc/nginx/sites-available/git.example.com.conf /etc/nginx/sites-enabled/

Remove the default config if not needed:

    sudo rm /etc/nginx/sites-enabled/default

#### Verify nginx configurations

    sudo nginx -t

#### Reload nginx service

    sudo systemctl reload nginx

#### Let's Encrypt for HTTPS

Certbot is now officially distributed via **snap**. The old PPA (`ppa:certbot/certbot`) and apt packages are deprecated and no longer maintained [2].

**1. Install snapd** (if not already installed):

    sudo apt install snapd
    sudo snap install core
    sudo snap refresh core

**2. Remove any old certbot packages:**

    sudo apt-get remove certbot python3-certbot-nginx -y

**3. Install Certbot via snap:**

    sudo snap install --classic certbot

**4. Ensure the certbot command is available:**

    sudo ln -s /snap/bin/certbot /usr/local/bin/certbot

**5. Obtain and install the certificate:**

    sudo certbot --nginx

Or specify domains directly:

    sudo certbot --nginx -d git1.example.com -d git2.example.com

Follow the on-screen instructions. Certbot will automatically configure HTTPS in your nginx config.

#### Verify auto-renewal

Certbot's snap package automatically installs a systemd timer that renews certificates before they expire [3]. No manual cron job or `--force-renewal` is needed.

Test that auto-renewal is working:

    sudo certbot renew --dry-run

You can verify the timer is active with:

    sudo systemctl list-timers | grep certbot

---

#### References

1. [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
2. [Certbot official installation instructions (snap)](https://certbot.eff.org/instructions?ws=nginx&os=snap)
3. [Let's Encrypt — Certbot snap announcement](https://community.letsencrypt.org/t/a-new-way-to-install-certbot-on-linux/120408)
