# 03-Catalogue Service

> **Hint** Catalogue Service manages products and categories. Built with Go, it stores data in MySQL and seeds product data on startup.

> **Dependency** MySQL must be running. Create the catalogue database and user before starting the service.

---

## Step 1: Install Go 1.22

```shell
dnf install -y golang git mysql8.4
go version
```

---

## Step 2: Setup Database

Download the artifact and run the SQL scripts against your MySQL server:

```shell
curl -L -o /tmp/catalogue.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/catalogue.zip
mkdir -p /app
cd /app
unzip /tmp/catalogue.zip
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/schema.sql
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/app-user.sql
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 catalogue < db/master-data.sql
```

> **Note** `schema.sql` creates the `catalogue` database and `products` table. `app-user.sql` creates the `catalogue` MySQL user. `master-data.sql` loads the 12 product records.

---

## Step 3: Add Application User, Create Directory, and Build Binary

```shell
useradd -r -s /bin/false appuser
cd /app
go mod tidy
CGO_ENABLED=0 go build -o /app/catalogue .
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

> **Hint** `CGO_ENABLED=0` creates a statically linked binary with no C dependencies.

---

## Step 4: Configure Systemd Service

Create the file `/etc/systemd/system/catalogue.service`:

```ini
[Unit]
Description=RoboShop Catalogue Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=/app/catalogue
Restart=on-failure
RestartSec=10
SyslogIdentifier=catalogue

Environment=MYSQL_HOST=localhost
Environment=MYSQL_USER=catalogue
Environment=MYSQL_PASSWORD=RoboShop@1
Environment=MYSQL_DATABASE=catalogue
Environment=PORT=8002

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` in `DB_HOST` with the private IP of the MySQL server.

---

## Step 5: Start and Verify

Enable and start the service:

```shell
systemctl daemon-reload
systemctl enable catalogue
systemctl start catalogue
```

Verify the service is running:

```shell
systemctl status catalogue
journalctl -u catalogue -f 
```
