# GUnicornExample (Video Link #4)
### **Step 1: Create and Configure a New User**

It is a best practice not to use the root user for configuration of servers.

* **Create a new user:**

```bash
sudo adduser <your_username>
```

* **Add the new user to the sudo group:**

```bash
sudo usermod -aG sudo <your_username>
```

* **Switch to the new user:**

```bash
su [-] <your_username>
```

---

### **Step 2: Firewall Configuration**

To see available applications, run:
```bash
sudo ufw app list
```

Set up the Uncomplicated Firewall (UFW) to allow SSH and later HTTP traffic.

* **Allow SSH and enable firewall, then verify status:**

```bash
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

---

### **Step 3: Update System and Install Dependencies**

* **Update package lists:**

```bash
sudo apt update
```

If you get error here open `sudo nano /etc/environment` and add `export GNUTLS_CPUID_OVERRIDE=0x1` and update again `sudo apt update`

* **Install Python, Virtual Environment, and Nginx:**

```bash
sudo apt install python3-dev python3-venv nginx libpq-dev curl
```

---

### **Step 4: Django Project Setup**

* **Create project workspace, create and activate a virtual environment:**

Creating separate virtual environments for each project is the industry standard because it isolates project-specific dependencies and prevents dependency conflicts/crushes by allowing projects to run on different Django versions and libraries without interfering with one another.

```bash
cd
mkdir DJangoWS && cd DJangoWS
mkdir testProj1 && cd testProj1
python3 -m venv env_testProj1
source env_testProj1/bin/activate
```

* **Install Django and Gunicorn:**

```bash
pip install django gunicorn
```

* **Create a Django project:**

```bash
django-admin startproject test_django .
```

* **Run migrations and collect static files:**

```bash
python manage.py migrate
python manage.py collectstatic
```

---

### **Step 5: Testing the Server**

To test if the Django project runs correctly:

* **Allow port 8000 and run the server:**

```bash
sudo ufw allow 8000
python manage.py runserver 0.0.0.0:8000
```

> **Note:** You may need to add your IP address to `ALLOWED_HOSTS` in `settings.py`.

---

### **Step 6: Configure Gunicorn Socket and Service**

You need to create a systemd socket and service file to manage Gunicorn.

* **Create the Gunicorn socket file:**

```bash
sudo nano /etc/systemd/system/gunicorn.socket
```

Add this content:

```ini
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

* **Create the Gunicorn service file:**

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add this content and replace `<your_username>` and project paths accordingly:

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

---

### **Step 7: Start Gunicorn**

* **Start and enable the socket:**

```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

* **Test with curl:**

```bash
curl --unix-socket /run/gunicorn.sock localhost
```

---

### **Step 8: Configure Nginx as a Reverse Proxy**

* **Create a new Nginx configuration file:**

```bash
sudo nano /etc/nginx/sites-available/test_django
```

Add this content and replace `<your_server_ip>` and `<your_username>` accordingly:

```nginx
server {
    listen 80;
    server_name <your_server_ip>;

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location /static/ {
        root /home/<your_username>/test_django;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

* **Enable the configuration and restart Nginx:**

```bash
sudo ln -s /etc/nginx/sites-available/test_django /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

---

### **Step 9: Final Cleanup**

* **Allow Nginx through the firewall and remove port 8000:**

```bash
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 8000
```

---
## **Tips:**
1. 3X Faster: https://blog.devops.dev/i-fixed-my-slow-nginx-gunicorn-setup-heres-how-it-became-3x-faster-1c324eb9bbb5
2. https://medium.com/@ganapriyakheersagar/hosting-django-application-with-nginx-and-gunicorn-in-production-99e64dc4345a
3. Best DJango Architecture Diagram: https://link.springer.com/content/pdf/10.1186/s40708-020-00103-3.pdf
4. https://mermaid.live/
5. https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu
6. https://link.springer.com/content/pdf/10.1186/s40708-020-00103-3.pdf

## **Video Links:**
1. https://www.youtube.com/watch?v=0roB7wZMLqI
2. https://www.youtube.com/playlist?list=PL-osiE80TeTtoQCKZ03TU5fNfx2UY6U4p
3. https://www.youtube.com/watch?v=tujhGdn1EMI
4. https://www.youtube.com/watch?v=NSHshIEVL-M
5. https://www.youtube.com/watch?v=QCDpBhW8XbM
6. https://www.youtube.com/watch?v=l0QEGvAX8rU
7. https://www.youtube.com/watch?v=_iUi8Sy6Muw
8. https://www.youtube.com/watch?v=dS_0eQno8i8

<img width="596" height="718" alt="django gunicorn nginx and MVT" src="https://github.com/user-attachments/assets/878139e7-1c50-4498-a1ef-2f4c93eb54ae" />
