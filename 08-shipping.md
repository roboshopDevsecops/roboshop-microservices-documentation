# 08-Shipping Service

> **Hint** Shipping Service provides city lookup and shipping cost calculation. Built with Java 21 + Spring Boot. It seeds city data automatically from bundled SQL on startup.

> **Dependency** MySQL must be running. Create the shipping database and user before starting.

---

## Step 1: Install Java 21

```shell
dnf install -y java-21-openjdk java-21-openjdk-devel maven mysql8.4
java -version
```

> **RHEL 10 Note** The application is developed with Java 21. RHEL 10 ships Java 21 and 25; Java 17 is not available.

---

## Step 2: Setup Database

Download the artifact and run the SQL scripts against your MySQL server:

```shell
curl -L -o /tmp/shipping.zip https://raw.githubusercontent.com/raghudevopsb89/roboshop-microservices/main/artifacts/shipping.zip
mkdir -p /app
cd /app
unzip /tmp/shipping.zip
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/schema.sql
mysql -h <MYSQL-SERVER-IP> -u root -pRoboShop@1 < db/app-user.sql
```

> **Note** City data (`data.sql`) is loaded automatically by Spring Boot on first startup via `spring.sql.init.mode=always`.

---

## Step 3: Build and Deploy

```shell
useradd -r -s /bin/false appuser
cd /app
mvn clean package -DskipTests
cp target/shipping.jar /app/shipping.jar
chown -R appuser:appuser /app
chmod o-rwx /app -R
```

---

## Step 4: Configure Systemd Service

Create the file `/etc/systemd/system/shipping.service`:

```ini
[Unit]
Description=RoboShop Shipping Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/app
ExecStart=java -jar /app/shipping.jar
Restart=on-failure
RestartSec=10
SyslogIdentifier=shipping

Environment=DB_HOST=localhost
Environment=DB_USER=shipping
Environment=DB_PASS=RoboShop@1
Environment=PORT=8004

[Install]
WantedBy=multi-user.target
```

> **Important** Replace `localhost` in `DB_HOST` with the private IP of the MySQL server.

---

## Step 5: Start and Verify

Enable and start the service:

```shell
systemctl daemon-reload
systemctl enable shipping
systemctl start shipping
```

Verify the service is running:

```shell
systemctl status shipping
journalctl -u shipping -f 
```
