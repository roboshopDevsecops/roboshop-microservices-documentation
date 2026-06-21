# 13-Ratings Service

> **Hint** Ratings Service stores product ratings submitted by users. Built with Python 3 / Flask. Stores data in MySQL.

> **Dependency** MySQL must be running. Create the ratings database and user before starting the service.

---

## Step 1 — Install Python 3

```shell
dnf install -y python3 python3-pip mysql8.4
```

---

## Step 2 — Setup Database

```shell
curl -L -o /tmp/ratings.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/ratings.zip
mkdir -p /app && cd /app
unzip /tmp/ratings.zip
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/schema.sql
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/app-user.sql
```

---

## Step 3 — Deploy Application

```shell
useradd -r -s /bin/false appuser
mkdir -p /app
pip3 install -r /app/requirements.txt cryptography
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 4 — Configure Systemd Service

Create `/etc/systemd/system/ratings.service`:

```ini
[Unit]
Description=RoboShop Ratings Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=gunicorn -b 0.0.0.0:8006 app:app
Restart=on-failure
RestartSec=10
SyslogIdentifier=ratings

Environment=MYSQL_HOST=localhost
Environment=MYSQL_USER=ratings
Environment=MYSQL_PASSWORD=RoboShop@1
Environment=MYSQL_DATABASE=ratings
Environment=PORT=8006

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` in `MYSQL_HOST` with the private IP of the MySQL server.

---

## Step 5 — Enable and Start

```shell
systemctl daemon-reload
systemctl enable ratings
systemctl start ratings
```

---

## Step 6 — Verify

```shell
systemctl status ratings
journalctl -u ratings -f
```

