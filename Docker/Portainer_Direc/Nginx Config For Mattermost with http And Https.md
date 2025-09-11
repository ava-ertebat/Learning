# Nginx Config For Mattermost with http And Https

Step 1: Preparing and Configuring Nginx
On Ubuntu, Nginx uses a very nice structure by default for managing multiple sites:

`/etc/nginx/sites-available/`: All configuration files for each site (subdomain) are placed here.

`/etc/nginx/sites-enabled/`: To enable a site, we create a symbolic link from its configuration file in sites-available to this directory.

This method allows us to easily enable or disable sites without deleting the main file.

First, let's make sure that the main Nginx file (nginx.conf) is configured to read configurations from the sites-enabled directory. This is done by default, but to be sure, run the following command:

1:First, we'll create a new file in the sites-available directory with the name of our subdomain. This will help keep the files organized.
```
sudo nano /etc/nginx/sites-available/chat.avacore.ir

```
Step 2: Configure the First Subdomain (Mattermost)
Now it’s time to tell Nginx how to treat the chat.avacore.ir subdomain. We’ll create a new configuration file that instructs Nginx to forward all incoming requests to this subdomain to the Mattermost container running on port 8065.

Create the configuration file:

First, we’ll create a new file in the sites-available directory with the name of our subdomain. This will help keep the files organized. Use the following command:
    listen 80;
    server_name chat.avacore.ir;

    location / {
        proxy_pass http://127.0.0.1:8065;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

`For Use with Https and Get Certificate Use This And Continio`

```
  GNU nano 6.2                                                             /etc/nginx/sites-available/chat.avacore.ir
# Temporary simplified config with WebSocket support
server {
    server_name chat.avacore.ir;

    location / {
        proxy_pass http://127.0.0.1:8065;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support headers
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/chat.avacore.ir/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/chat.avacore.ir/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = chat.avacore.ir) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name chat.avacore.ir;
    return 404; # managed by Certbot


}



```
3. Site activation:
Now we need to enable this configuration by creating a symbolic link in sites-enabled:

```
sudo ln -s /etc/nginx/sites-available/chat.avacore.ir /etc/nginx/sites-enabled/

```
4. Test and Apply the Configuration:

Before finalizing, it is always best to test the Nginx configuration to ensure there are no errors:

```
sudo nginx -t

```
5.If you see the output syntax is ok and test is successful, it means everything is correct. Now apply the settings with the following command:

```
sudo systemctl reload nginx

```

6.It's time to test!
Now open your browser and go to http://chat.avacore.ir. Do you see the Mattermost page? (You may be redirected to HTTPS due to Mattermost settings, which is giving an error for now, this is normal. The important thing is to know that the initial connection has been established.)

I'm waiting for your result.

7.Step 3: Secure with SSL/TLS (HTTPS)
In this step, we will use a tool called Certbot to automatically obtain a free SSL certificate from Let's Encrypt and install it on Nginx. This will encrypt all traffic between your users and your server.

1. Install Certbot:

First, we will install Certbot and its Nginx plugin with the following commands. This plugin allows Certbot to automatically read and modify your Nginx configuration files.

```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

```
2. Obtain and install an SSL certificate:

Now we run the main command. This command tells Certbot to issue a certificate for the domain chat.avacore.ir using the Nginx plugin:

```
sudo certbot --nginx -d chat.avacore.ir

```

What will happen?
After running this command, Certbot will ask you a few questions:

Email address: To send important notifications (such as approaching certificate expiration date).

Accept Terms of Service: You must agree to it.

Subscribe to newsletter: This is optional.

Redirect HTTP to HTTPS: It will ask you if you want all HTTP traffic to be automatically redirected to HTTPS. Be sure to select the Redirect option. This increases security.

Certbot will then automatically edit your `/etc/nginx/sites-available/chat.avacore.ir` configuration file, add the certificate path to it, and reload Nginx.

Time for the final test!

Run these commands and after successful completion, open your browser and go to `https://chat.avacore.ir` (this time with https). You should see the Mattermost page with a green or gray lock in your browser's address bar, indicating that the connection is secure.
