# **Nginx Installation and Configuration for example.com** 🌐🔒
<p align="justify">
This comprehensive guide will help you install, configure, and secure **Nginx** for your domain **example.com**. It includes detailed instructions on setting up server blocks, enabling HTTPS with Let’s Encrypt, and configuring Nginx as a reverse proxy. We’ll also cover optional steps to create a web root directory and test your server.
</p>

---

## **Table of Contents** 📑

1. [**Introduction**](#introduction-) 📝  
2. [**Steps to Install and Configure Nginx**](#steps-to-install-and-configure-nginx-) 🚀  
   - 2.1 [Update System Packages](#21-update-system-packages-) 📦  
   - 2.2 [Install Nginx](#22-install-nginx-) 🔧  
   - 2.3 [Start and Enable Nginx](#23-start-and-enable-nginx-) 🔄  
   - 2.4 [Configure Firewall to Allow Nginx](#24-configure-firewall-to-allow-nginx-) 🔥  
   - 2.5 [Create Nginx Server Block](#25-create-nginx-server-block-%EF%B8%8F) 🖥️  
   - 2.6 [Enable the Server Block](#26-enable-the-server-block-) 🔗  
   - 2.7 [Test Nginx Configuration](#27-test-nginx-configuration-) 🧪  
   - 2.8 [Reload Nginx](#28-reload-nginx-) 🔁  
   - 2.9 [Test Your Domain](#29-test-your-domain-) 🌐  
3. [**SSL Configuration with Let’s Encrypt**](#ssl-configuration-with-lets-encrypt-) 🔒  
   - 3.1 [Install Certbot and Nginx Plugin](#install-certbot-and-nginx-plugin-) 🔑  
   - 3.2 [Obtain an SSL Certificate](#obtain-an-ssl-certificate-) 🛡️  
   - 3.3 [Test Certificate Renewal](#test-certificate-renewal-) 🔄  
   - 3.4 [Reload Nginx with SSL Configuration](#reload-nginx-with-ssl-configuration-) 🔁  
4. [**Optional Steps**](#optional-steps-) 🧑‍💻  
   - 4.1 [Create the Web Root Directory](#create-the-web-root-directory-) 🏁  
   - 4.2 [Add a Test HTML Page](#add-a-test-html-page-) 📝  
5. [**Conclusion**](#conclusion-) 🎉

---

## **Introduction** 📝

In this guide, we will walk you through setting up **Nginx** as a reverse proxy for **example.com**. The instructions will cover the following:

- Installing and configuring **Nginx**.
- Setting up a server block for **example.com**.
- Configuring **Nginx** as a reverse proxy to forward requests to a backend server.
- Securing your domain with **HTTPS** using **Let’s Encrypt**.
- Optionally, creating a web root directory and serving a test page to verify the setup.

Let’s dive into the installation and configuration steps! 🚀

---

## **Steps to Install and Configure Nginx** 🚀

### 2.1 **Update System Packages** 📦

Before you begin, it’s essential to update the system to ensure all packages are up to date. This step will make sure your Nginx installation is smooth.

```bash
sudo apt update
sudo apt upgrade -y
```

### 2.2 **Install Nginx** 🔧

Now, you’re ready to install **Nginx**. Run the following command to install it using your package manager.

```bash
sudo apt install nginx -y
```

### 2.3 **Start and Enable Nginx** 🔄

Once Nginx is installed, you need to start it and ensure it starts automatically upon system boot.

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2.4 **Configure Firewall to Allow Nginx** 🔥

If you’re using **UFW (Uncomplicated Firewall)**, allow HTTP and HTTPS traffic. This will enable external access to your Nginx web server.

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

### 2.5 **Create Nginx Server Block** 🖥️

A **server block** (virtual host) tells Nginx how to handle requests for your domain. Let’s create one for **example.com**.

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Add the following configuration to the file:

```nginx
server_tokens               off;
access_log                  /var/log/nginx/example.com.access.log;
error_log                   /var/log/nginx/example.com.error.log;

server {
    listen 80;
    server_name example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl; # managed by Certbot
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_pass http://localhost:8000;

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

This configuration does the following:
- **HTTP to HTTPS Redirect**: All HTTP traffic is redirected to HTTPS.
- **SSL Configuration**: Sets up SSL with certificates obtained via **Let’s Encrypt**.
- **Reverse Proxy**: Forwards incoming requests to the backend running on **localhost:8000**.

### 2.6 **Enable the Server Block** 🔗

After creating the server block, you need to enable it by creating a symbolic link.

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

### 2.7 **Test Nginx Configuration** 🧪

Before reloading Nginx, test the configuration for syntax errors.

```bash
sudo nginx -t
```

### 2.8 **Reload Nginx** 🔁

Reload Nginx to apply the new configuration.

```bash
sudo systemctl reload nginx
```

### 2.9 **Test Your Domain** 🌐

Ensure your domain **example.com** is pointing to your server’s public IP. Test your setup by visiting **http://example.com** in a web browser.

---

## **SSL Configuration with Let’s Encrypt** 🔒

### 3.1 **Install Certbot and Nginx Plugin** 🔑

To secure your domain with HTTPS, you’ll need **Certbot** and the Nginx plugin to obtain and configure SSL certificates.

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

### 3.2 **Obtain an SSL Certificate** 🛡️

Now that Certbot is installed, obtain a free SSL certificate for your domain.

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot will automatically configure SSL for Nginx.

### 3.3 **Test Certificate Renewal** 🔄

To ensure that your certificate will be automatically renewed before it expires, run the following dry run:

```bash
sudo certbot renew --dry-run
```

### 3.4 **Reload Nginx with SSL Configuration** 🔁

Reload Nginx again to apply the SSL configuration.

```bash
sudo systemctl reload nginx
```

---

## **Optional Steps** 🧑‍💻

### 4.1 **Create the Web Root Directory** 🏁

If you plan to serve static files, create a directory for your web root.

```bash
sudo mkdir -p /var/www/example.com
sudo chown -R $USER:$USER /var/www/example.com
sudo chmod -R 755 /var/www/example.com
```

### 4.2 **Add a Test HTML Page** 📝

Verify your configuration by adding a simple test HTML page:

```bash
echo '<html><head><title>Welcome to example.com</title></head><body><h1>Success! Nginx is serving example.com</h1></body></html>' | sudo tee /var/www/example.com/index.html
```

---

## **Conclusion** 🎉

Congratulations! You’ve successfully installed and configured **Nginx** for your domain **example.com**, secured it with **Let’s Encrypt**, and set up a reverse proxy. Optional steps like creating a web root directory and adding a test HTML page will help you verify the setup.

With **Nginx** now running securely on your server, your website is ready to handle traffic efficiently! 😊🌟
