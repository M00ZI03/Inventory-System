# Installation guide (Linux Mint / Ubuntu)

Tested on a fresh Linux Mint install with 3 GB of RAM. Commands assume a regular user with `sudo`.

## What you need

- A PC running Linux Mint or Ubuntu, connected to your network
- Git and Node.js 20 or newer (only needed on the machine where you build the app)
- About 15 minutes

---

## Step 1: Update the system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git nginx unzip curl ufw
```

## Step 2: Install PocketBase

1. Go to the PocketBase releases page (github.com/pocketbase/pocketbase/releases) and copy the link for the **Linux amd64** zip.
2. Download and unzip it:

```bash
mkdir -p ~/stock-app/pocketbase && cd ~/stock-app/pocketbase
curl -L -o pb.zip "PASTE_THE_LINUX_AMD64_LINK_HERE"
unzip pb.zip && rm pb.zip
chmod +x pocketbase
```

## Step 3: Start PocketBase (bound to this machine only)

```bash
./pocketbase serve --http=127.0.0.1:8090
```

Open `http://127.0.0.1:8090/_/` in a browser on this PC and create your **admin account**. Then stop it with `Ctrl + C`. We will run it as a service in Step 6.

## Step 4: Create the collections

In the PocketBase dashboard, create the collections the app uses (for example `items`) with the same field names the code expects. If this repository includes a `pb_schema.json`, import it instead from **Settings, then Import collections**.

## Step 5: Build and install the app

On any computer with Node.js (a faster machine is best):

```bash
git clone https://github.com/M00ZI03/YOUR-REPO-NAME.git
cd YOUR-REPO-NAME
cp .env.example .env
npm install
npm run build
```

Copy the generated `dist/` folder to the server:

```bash
sudo mkdir -p /var/www/stock-app
sudo cp -r dist/* /var/www/stock-app/
```

## Step 6: Run PocketBase as a service

Create the service file (replace `YOUR_USER` with your Linux username):

```bash
sudo nano /etc/systemd/system/pocketbase.service
```

Paste:

```ini
[Unit]
Description=PocketBase
After=network.target

[Service]
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER/stock-app/pocketbase
ExecStart=/home/YOUR_USER/stock-app/pocketbase/pocketbase serve --http=127.0.0.1:8090
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pocketbase
sudo systemctl status pocketbase
```

## Step 7: Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/stock-app
```

Paste:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/stock-app;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/stock-app /etc/nginx/sites-enabled/stock-app
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```

## Step 8: Open the firewall for the web server only

```bash
sudo ufw allow 80/tcp
sudo ufw allow ssh
sudo ufw enable
```

Do **not** open port 8090. PocketBase should stay private.

## Step 9: Open the app

Find the server's address:

```bash
hostname -I
```

On any phone or laptop on the same WiFi, open `http://THAT_ADDRESS/`.

## Step 10 (optional): Remote access with NetBird

1. Install NetBird on the server and on each device you want to use remotely, then sign in to the same NetBird network.
2. From a remote device, open `http://SERVER_NETBIRD_ADDRESS/`.

No router ports need to be opened.

## Backups

Everything lives in the `pb_data` folder next to the PocketBase binary. Stop the service and copy that folder to back it up:

```bash
sudo systemctl stop pocketbase
cp -r ~/stock-app/pocketbase/pb_data ~/pb_data_backup_$(date +%F)
sudo systemctl start pocketbase
```

## Updating the app

Pull the latest code, run `npm run build`, and copy the new `dist/` contents to `/var/www/stock-app/`.

## Troubleshooting

| Problem | Fix |
|---|---|
| Page loads but data does not | Check `sudo systemctl status pocketbase` and that `/api/` is in the Nginx config |
| Blank page after refresh on a sub-page | Make sure the `try_files $uri /index.html;` line is present |
| Cannot open the app from a phone | Confirm both devices are on the same WiFi and `ufw` allows port 80 |
| Build crashes on the server | Build on another computer and copy `dist/` over |
