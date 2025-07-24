# 🚀 Manual Deployment of Django Application on AWS EC2 (Ubuntu)

This document outlines the step-by-step **manual deployment** of a Django web application on an **EC2 Ubuntu instance**, using **Gunicorn** as the WSGI server and **Nginx** as a reverse proxy.

---

## 🔧 Tech Stack Used

- **Amazon EC2** (Ubuntu)
- **Django 4.x**
- **Gunicorn** (WSGI server)
- **Nginx** (reverse proxy)
- **Python 3.10**
- **Git + GitHub**
- **PuTTY (.ppk) for SSH access**
- **Virtualenv**

---

## 🪜 Deployment Steps

### 1. **Connect to EC2 using PuTTY**
- Use `.ppk` key with user: `ubuntu@<your-ec2-public-ip>`

---

### 2. **Update & Install Required Packages**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3.10-venv python3-pip git nginx -y
```

---

### 3. **Clone Your Django Project**
```bash
git clone https://github.com/kaustubh2949/django_repo.git
cd django_repo
```

---

### 4. **Create & Activate Virtual Environment**
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

---

### 5. **Set ALLOWED_HOSTS in settings.py**
```python
# In employee_pro/settings.py
ALLOWED_HOSTS = ['<your-ec2-ip>', 'localhost']
```

---

### 6. **Run Migrations & Collect Static Files**
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic --noinput
```

---

### 7. **Test with Django Dev Server**
```bash
python manage.py runserver 0.0.0.0:8000
```
- Visit: `http://<your-ec2-ip>:8000`

---

### 8. **Install and Run Gunicorn**
```bash
pip install gunicorn
gunicorn --bind 0.0.0.0:8000 employee_pro.wsgi:application
```

---

### 9. **Set Up Gunicorn as a Systemd Service**
Create file:
```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Paste:
```ini
[Unit]
Description=gunicorn daemon for Django
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/django_repo
ExecStart=/home/ubuntu/django_repo/venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/django_repo/gunicorn.sock employee_pro.wsgi:application

[Install]
WantedBy=multi-user.target
```

Then run:
```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

---

### 10. **Configure Nginx**
```bash
sudo nano /etc/nginx/sites-available/django
```

Paste:
```nginx
server {
    listen 80;
    server_name <your-ec2-ip>;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/django_repo;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/django_repo/gunicorn.sock;
    }
}
```

Enable and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/django /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

---

### 11. **Open Ports in EC2 Security Group**
- Open port **80** (HTTP)
- (If testing with dev server: port **8000**)

---

### ✅ Final Result

Your Django app is now live at:
```
http://<your-ec2-public-ip>
```

---

## 💡 Challenges Faced

- Gunicorn socket permission issues
- Nginx proxy path misconfig
- Static file collection path
- Port 80 blocked: fixed via Security Group

---

## 📘 Learnings

- How WSGI servers and reverse proxies work
- Manual service setup on Linux using systemd
- Hands-on with Nginx and socket-based deployment
- Environment isolation using `venv`

---

✅ **Status**: Deployed successfully using manual setup (Gunicorn + Nginx + EC2 Ubuntu)

