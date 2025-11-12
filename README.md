<!---
{
  "id": "602e52e7-553e-40a0-a6f6-3ea0c4768835",
  "teaches": "Serving Local Directories with Nginx and Server Blocks",
  "depends_on": ["b9e34253-61e2-4fb7-bd36-388c370fa765"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-06",
  "keywords": ["nginx", "localhost", "virtualhosts", "subdomains", "webserver"]
}
--->

# Serving Local Directories with Nginx and Server Blocks

> In this exercise you will learn how to configure the Nginx web server to serve local directories on `127.0.0.1`. Furthermore we will explore how to use server blocks to simulate subdomains locally by editing your system's hosts file and defining multiple local configurations.

### Introduction

Nginx (pronounced "engine-x") is a high-performance web server that is widely used for serving static content, acting as a reverse proxy, and balancing loads. Known for its speed and scalability, it is commonly used in production environments to serve millions of requests per day. However, it also excels as a lightweight local development server.

In local development, it's often useful to simulate a full web hosting environment. By configuring Nginx to bind to `127.0.0.1`, you can safely test sites without exposing them to the public internet. With multiple server blocks—Nginx's equivalent of Apache's virtual hosts—you can serve different content under different domain names, even subdomains, all from your local machine.

This exercise walks you through installing Nginx, configuring it to serve directories from `127.0.0.1`, and setting up multiple named server blocks to simulate subdomains. You will modify system configuration files and directory permissions, so administrative access is required. This setup will be valuable for frontend development, microservices testing, or mimicking deployment environments.

### Further Readings and Other Sources

* [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
* [DigitalOcean Guide: How To Set Up Nginx Server Blocks](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-ubuntu-20-04)
* [YouTube: Nginx Virtual Hosts Setup](https://www.youtube.com/watch?v=ZFY9y8UXojU)
* DOI: [10.1007/978-1-4842-4470-1\_8](https://doi.org/10.1007/978-1-4842-4470-1_8)

---

## Tasks

### Task 1: Install Nginx

1. Update your package list:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Install Nginx:

   ```bash
   sudo apt install nginx
   ```

3. Verify installation:

   ```bash
   nginx -v
   sudo systemctl status nginx
   ```

### Task 2: Configure Nginx to Bind to 127.0.0.1

1. Open the default site config:

   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

2. Modify the `listen` directive:

   ```nginx
   listen 127.0.0.1:80 default_server;
   ```

3. Restart Nginx to apply changes:

   ```bash
   sudo systemctl restart nginx
   ```

### Task 3: Serve a Local Directory

1. Create a local directory:

   ```bash
   mkdir -p ~/websites/project1
   echo "<h1>Hello from Project1</h1>" > ~/websites/project1/index.html
   ```

2. Create a new server block:

   ```bash
   sudo nano /etc/nginx/sites-available/project1.local
   ```

3. Paste this configuration:

   ```nginx
   server {
       listen 127.0.0.1:80;
       server_name project1.local;

       root /home/YOUR_USERNAME/websites/project1;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

   * `listen`: binds the server to the local loopback address.
   * `server_name`: the domain name to match.
   * `root`: path to the served directory.
   * `location /`: defines how requests to `/` are handled.

4. Enable the config:

   ```bash
   sudo ln -s /etc/nginx/sites-available/project1.local /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. Add to your hosts file:

   ```bash
   sudo nano /etc/hosts
   ```

   Add this line:

   ```
   127.0.0.1 project1.local
   ```

6. Visit [http://project1.local](http://project1.local) in a browser.

### Task 4: Serve Subdomains with Server Blocks

1. Create a new directory:

   ```bash
   mkdir -p ~/websites/api.project1
   echo "<h1>Hello from API</h1>" > ~/websites/api.project1/index.html
   ```

2. Create a new server block config:

   ```bash
   sudo nano /etc/nginx/sites-available/api.project1.local
   ```

3. Paste the config:

   ```nginx
   server {
       listen 127.0.0.1:80;
       server_name api.project1.local;

       root /home/YOUR_USERNAME/websites/api.project1;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

4. Enable it and update hosts:

   ```bash
   sudo ln -s /etc/nginx/sites-available/api.project1.local /etc/nginx/sites-enabled/
   echo "127.0.0.1 api.project1.local" | sudo tee -a /etc/hosts
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. Open [http://api.project1.local](http://api.project1.local) to verify it works.

---

## Advice

Nginx is both simple and powerful once you're familiar with its configuration style. Always validate your configuration with `nginx -t` before restarting the service to catch syntax errors early. Local subdomain setups like these mirror production environments and allow seamless testing of microservices, routing, or content separation strategies. Be cautious with file permissions and double-check paths—Nginx won’t serve directories it can’t read. Once this setup is complete, consider exploring HTTPS using self-signed certificates or reverse proxy configurations. This foundational skill will greatly aid in local development and deployment simulations.
