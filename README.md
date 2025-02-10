Here’s a detailed and emoji-enhanced guide for installing and configuring Nginx for the domain **phidashexport.com**:

---

### **Description** 📜

This guide will walk you through the process of setting up Nginx for the domain **phidashexport.com**, including:
1. Installing and configuring Nginx.
2. Setting up server blocks (virtual hosts).
3. Configuring Nginx as a reverse proxy.
4. Securing your site with HTTPS using **Let's Encrypt**.

Additionally, optional steps for creating a web root directory and adding a test page are included to ensure everything is working as expected.

---

### **Steps to Install and Configure Nginx** 🚀

#### 1. **Update System Packages** 📦
It’s always good practice to ensure your system is up to date before installing new packages.
```bash
sudo apt update
sudo apt upgrade -y
```

#### 2. **Install Nginx** 🔧
Now, install Nginx using your package manager. This step will set up Nginx on your server.
```bash
sudo apt install nginx -y
```

#### 3. **Start and Enable Nginx** 🔄
Start Nginx and ensure it starts automatically on boot.
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### 4. **Configure Firewall to Allow Nginx** 🔥
If you’re using **UFW (Uncomplicated Firewall)**, allow Nginx traffic. This ensures that HTTP (80) and HTTPS (443) traffic can reach your server.
```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

#### 5. **Create Nginx Server Block** 🖥️
Create the server block configuration file for your domain. This file defines how Nginx serves requests for **phidashexport.com**.
```bash
sudo nano /etc/nginx/sites-available/phidashexport.com
```
Add the following configuration:
```nginx
server_tokens               off;
access_log                  /var/log/nginx/phidashexport.access.log;
error_log                   /var/log/nginx/phidashexport.error.log;

server {
    listen 80;
    server_name phidashexport.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl; # managed by Certbot
    server_name phidashexport.com;

    ssl_certificate /etc/letsencrypt/live/phidashexport.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/phidashexport.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_pass http://localhost:9000;

        # Timeouts for reverse proxy connection
        proxy_read_timeout 300s;  # Increase this value as needed
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;        

        # Headers for reverse proxy setup
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Keep-Alive "timeout=600";
    }
}
```
This configuration:
- Redirects HTTP to HTTPS.
- Configures SSL for secure communication.
- Sets up Nginx as a reverse proxy to forward requests to your backend on **localhost:9000**.

#### 6. **Enable the Server Block** 🔗
Create a symbolic link to enable the server block.
```bash
sudo ln -s /etc/nginx/sites-available/phidashexport.com /etc/nginx/sites-enabled/
```

#### 7. **Test Nginx Configuration** 🧪
Ensure there are no syntax errors in the Nginx configuration.
```bash
sudo nginx -t
```

#### 8. **Reload Nginx** 🔄
Reload Nginx to apply the configuration changes.
```bash
sudo systemctl reload nginx
```

#### 9. **Test Your Domain** 🌐
Ensure your domain **phidashexport.com** is pointing to your server’s public IP in your DNS settings. Visit **http://phidashexport.com** to verify the setup.

#### 10. **Install Certbot and Nginx Plugin** 🔒
Install **Certbot** and the Nginx plugin to obtain an SSL certificate.
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

#### 11. **Obtain an SSL Certificate** 🛡️
Use Certbot to obtain a free SSL certificate from **Let’s Encrypt**.
```bash
sudo certbot --nginx -d phidashexport.com -d www.phidashexport.com
```
Certbot will automatically configure SSL for you.

#### 12. **Test Certificate Renewal** 🔄
Check if Certbot will renew your certificate automatically before it expires.
```bash
sudo certbot renew --dry-run
```

#### 13. **Reload Nginx** 🔁
Reload Nginx again to apply the SSL configuration.
```bash
sudo systemctl reload nginx
```

#### 14. **(Optional) Create the Web Root Directory** 🏁
If you need a place to serve static files (e.g., HTML, images), create a web root directory:
```bash
sudo mkdir -p /var/www/phidashexport.com
sudo chown -R $USER:$USER /var/www/phidashexport.com
sudo chmod -R 755 /var/www/phidashexport.com
```

#### 15. **(Optional) Add a Test HTML Page** 📝
To test that everything is working, add a simple HTML page.
```bash
echo '<html><head><title>Welcome to Phidashexport</title></head><body><h1>Success! Nginx is serving phidashexport.com</h1></body></html>' | sudo tee /var/www/phidashexport.com/index.html
```

---

### **Conclusion** 🎉

By following these steps, you will have successfully installed Nginx, configured it as a reverse proxy, secured your site with an SSL certificate from Let’s Encrypt, and tested the entire setup. Optional steps allow you to serve static content and test your configuration with a sample HTML page.

Enjoy your secure and optimized Nginx setup for **phidashexport.com**! 😊
