# 02-MySQL

> **Hint** MySQL stores structured relational data for the Catalogue, Shipping, Ratings, and Orders services. Each service has its own dedicated database and user — no services share a database.

---

## Install

Install the MySQL server package and enable it to start on boot:

```shell
dnf install -y mysql8.4-server
systemctl enable mysqld
systemctl start mysqld
```

> **RHEL 10 Note** The package is named mysql8.4-server (not mysql-server). RHEL 10 ships MySQL 8.4 from the AppStream repository.


---

## Configure

### Set Root Password

On RHEL 10, MySQL starts with the root@localhost user using auth_socket (no password). Set a password for root access:

```shell
mysql -u root -e "
  CREATE USER 'root'@'%' IDENTIFIED BY 'RoboShop@1';
  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'RoboShop@1';
  FLUSH PRIVILEGES;
"
```


---

Confirm you can connect as root and list databases:

```shell
mysql -u root -pRoboShop@1 -e "SHOW DATABASES;"
```


