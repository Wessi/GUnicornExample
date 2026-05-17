sudo usermod -aG sudo <your_username>
    ```
*   **Switch to the new user:**
    
```bash
    su - <your_username>
    ```

### **Step 2: Firewall Configuration**
Set up the Uncomplicated Firewall (UFW) to allow SSH and later HTTP traffic.
*   **Allow SSH and enable firewall:**
    
```bash
    sudo ufw allow OpenSSH
    sudo ufw enable
    sudo ufw status
    ```

### **Step 3: Update System and Install Dependencies**
*   **Update package lists:**
    ```bash
    sudo apt update
    ```
*   **Install Python, Virtual Environment, and Nginx:**
    ```bash
    sudo apt install python3-pip python3-dev python3-venv nginx libpq-dev
    ```

### **Step 4: Django Project Setup**
*   **Create and activate a virtual environment:**
    
```bash
    python3 -m venv env
    source env/bin/activate
    ```
*   **Install Django and Gunicorn:**
    ```bash
    pip install django gunicorn
    ```
*   **Create a Django project:**
    ```bash
    django-admin startproject test_django .
    ```
*   **Run migrations and collect static files:**
    
```bash
    python manage.py migrate
    python manage.py collectstatic
    ```

### **Step 5: Testing the Server**
To test if the Django project runs correctly:
*   **Allow port 8000 and run the server:**
    
```bash
    sudo ufw allow 8000
    python manage.py runserver 0.0.0.0:8000
    ```
    *(Note: You may need to add your IP to `ALLOWED_HOSTS` in `settings.py` [[07:44](https://www.youtube.com/watch?v=NSHshIEVL-M&t=464)])*

### **Step 6: Configure Gunicorn Socket and Service**

You need to create a systemd socket and service file to manage Gunicorn.
*   **Create the Gunicorn Socket file:**
    
```bash
    sudo nano /etc/systemd/system/gunicorn.socket
    ```
    *Add this content:*
    
```ini
    [Unit]
    Description=gunicorn socket

    [Socket]
    ListenStream=/run/gunicorn.sock

    [Install]
    WantedBy=sockets.target
    ```
*   **Create the Gunicorn Service file:**
    
```bash
    sudo nano /etc/systemd/system/gunicorn.service
    ```
    *Add this content (replace `<username>` and project paths accordingly):*
    
```ini
    [Unit]
    Description=gunicorn daemon
    Requires=gunicorn.socket
    After=network.target

    [Service]
    User=<your_username>
    Group=www-data
    WorkingDirectory=/home/<your_username>/test_django
    ExecStart=/home/<your_username>/env/bin/gunicorn \
              --access-logfile - \
              --workers 3 \
              --bind unix:/run/gunicorn.sock \
              test_django.wsgi:application

    [Install]
    WantedBy=multi-user.target
    ```

### **Step 7: Start Gunicorn**
*   **Start and enable the socket:**
    
```bash
    sudo systemctl start gunicorn.socket
    sudo systemctl enable gunicorn.socket
    ```
*   **Test with Curl:**
    
```bash
    curl --unix-socket /run/gunicorn.sock localhost
    ```

### **Step 8: Configure Nginx as a Reverse Proxy**
*   **Create a new Nginx configuration file:**
    ```bash
    sudo nano /etc/nginx/sites-available/test_django
    ```
    *Add this content (replace with your server IP):*
    
```nginx
    server {
        listen 80;
        server_name <your_server_ip>;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /home/<your_username>/test_django;
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
    }
    ```
*   **Enable the configuration and restart Nginx:**
    
```bash
    sudo ln -s /etc/nginx/sites-available/test_django /etc/nginx/sites-enabled
    sudo nginx -t
    sudo systemctl restart nginx
    ```

### **Step 9: Final Cleanup**
*   **Allow Nginx through the firewall and remove port 8000:**
    ```bash
    sudo ufw allow 'Nginx Full'
    sudo ufw delete allow 8000
    ```

Your Django application should now be live at your server's IP address!