# ACIT 3475 Project Part 2 - Jovica Kuzmanovic

This project extends a personal portfolio website by adding secure authentication, GitHub contributions display, and high-availability architecture using load balancing.

---

## 🧩 Part 1: OAuth Integration and GitHub Contributions

### ✅ OAuth Integration

- Implemented **Google OAuth 2.0** for user login.
- Only authenticated users can access and edit the **Projects** section.
- OAuth tokens are handled securely on the backend and not exposed to the frontend.

**Technologies Used:**
- Google Cloud Console for OAuth credentials
- Flask 

---

### 📅 GitHub Contributions Widget

- Embedded the GitHub contributions calendar using:
- Customized styling to match the portfolio theme.
- Shows the user’s real-time GitHub activity to enhance credibility.


## Prepartion
We need to setop 3 AWS EC2 Instances Setup, with availability on Ports 22(ssh) and HTTP(80).


## Step 1: Nginx Reverse Proxy Setup for HTTPS Backend

### ⚖️ Task:

Configure an EC2 instance running Nginx to reverse proxy requests to an external HTTPS backend: `https://yovitsa-kuzmanovic.site/`

### Process:

1. Install Nginx

   ```bash
   sudo apt update
   sudo apt install -y nginx
   ```

2. Create Reverse Proxy Config

   ```bash
   sudo vim /etc/nginx/sites-available/reverse_proxy.conf
   ```

3. Paste the following config:

   ```nginx
   server {
       listen 80;

       location / {
           proxy_pass https://yovitsa-kuzmanovic.site/;
           proxy_ssl_server_name on;
           proxy_ssl_verify off;

           proxy_set_header Host yovitsa-kuzmanovic.site;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. Enable the config

   ```bash
   sudo ln -s /etc/nginx/sites-available/reverse_proxy.conf /etc/nginx/sites-enabled/
   sudo rm /etc/nginx/sites-enabled/default
   ```

5. Test and restart Nginx

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

6. Test the reverse proxy
   ```bash
   curl http://localhost
   ```

### 📈 Result:

The portfolio page content returned by `curl`

## Step 2: HAProxy Load Balancer Setup for Two Nginx Reverse Proxies

### ⚖️ Task:

Set up HAProxy on a third EC2 instance to load balance traffic between two existing Nginx reverse proxy servers.

### 🧪 Process:

1. **Install HAProxy**

   ```bash
   sudo apt update
   sudo apt install -y haproxy
   ```

2. **Edit the HAProxy config**

   ```bash
   sudo nano /etc/haproxy/haproxy.cfg
   ```

3. **Replace or append the config with:**

   ```haproxy
   frontend http_front
       bind *:80
       default_backend aws_backends

   backend aws_backends
       balance roundrobin
       server nginx1 172.31.29.80:80 check
       server nginx2 172.31.26.80:80 check
   ```

4. **Restart HAProxy**

   ```bash
   sudo systemctl restart haproxy
   ```

5. **Test the Load Balancer**

   ```bash
   curl http://localhost
   curl http://<HAProxy_Public_IP>
   ```

## 🌐 Part 3: Scalability, Performance, and Documentation

### ⚙️ High Availability Testing

- Used **Apache Benchmark (ab)** to simulate 1000 requests with 100 concurrent users:

To install Apache Benchmark run the command below:
   
   
   ```bash
   sudo apt update
   sudo apt install apache2-utils
   ```
   Then we need to run:
   
   ```bash
   ab -n 1000 -c 100 http://<haproxy-public-ip>/
   ```
    
    

On the images below you can see the benchmarking results:

### 📈 Test 1
![alt text](/Bechmarking_1.png)

### 📈 Test 2
![alt text](/Bechmarking_2.png)
##  Tech Stack
- Flask (Python Backend)

- Nginx (Reverse Proxy)

- HAProxy (Load Balancer)

- Google OAuth

- GitHub Calendar Widget

- Apache Benchmark for Testing