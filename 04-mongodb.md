# 04-MongoDB

> **Hint** MongoDB stores user profiles and order documents as flexible JSON-like documents. The User Service and Orders Service both depend on this server being reachable before they can start.

---

## Install

### Add the MongoDB 7.0 Repository

Create the repo file at `/etc/yum.repos.d/mongodb-org-7.0.repo`:

```ini
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/7.0/x86_64/
gpgcheck=0
enabled=1
```

> **RHEL 10 Note** GPG checking is disabled because the MongoDB 7.0 signing key is not recognized on RHEL 10. The RHEL 9 packages are used as MongoDB does not yet provide native RHEL 10 packages.

### Install the Package

```shell
dnf install -y mongodb-org
```

### Enable and Start

```shell
systemctl enable mongod
systemctl start mongod
```

---

## Configure

By default MongoDB binds only to `127.0.0.1`. Because the User Service and Orders Service run on separate servers, MongoDB must accept remote connections.

Edit `/etc/mongod.conf` and change the `bindIp` line under the `net` section:

```yaml
net:
  bindIp: 0.0.0.0
```

> **Important** Binding to `0.0.0.0` exposes MongoDB on all interfaces. In production, restrict access using firewall rules or a VPC security group so that only the application servers can reach port 27017.

Restart the service to apply the change:

```shell
systemctl restart mongod
```

---

## Start

MongoDB is enabled and started during the Install step. To restart manually if needed:

```shell
systemctl restart mongod
```

