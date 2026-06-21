# RoboShop Microservices

> **Documentation & deployment guide** for the RoboShop platform.  
> **Full source code, CI/CD, and infrastructure** live in the [**roboshopDevsecops**](https://github.com/roboshopDevsecops) GitHub organization.

## Quick Links

| Resource | Link |
|----------|------|
| **Organization (all repos)** | [github.com/roboshopDevsecops](https://github.com/roboshopDevsecops) |
| **Microservices source code** | See [Source Code Repositories](#source-code-repositories) below |
| **Terraform / Kubernetes (v2)** | [roboshop-v2](https://github.com/roboshopDevsecops/roboshop-v2) |
| **Helm charts (v1)** | [roboshop-helm-v1](https://github.com/roboshopDevsecops/roboshop-helm-v1) |
| **CI/CD workflows** | [github-reusable-workflows](https://github.com/roboshopDevsecops/github-reusable-workflows) |
| **Observability stack** | [ai-obs](https://github.com/roboshopDevsecops/ai-obs) |
| **Load testing** | [load-test](https://github.com/roboshopDevsecops/load-test) |

## Application Description

RoboShop Microservices is a cloud-native e-commerce application for robotics components. It consists of 9 services written in 5 programming languages, backed by 4 database types, with a Next.js static frontend served by Nginx.

This repository contains the **architecture overview**, **13-step VM deployment guides**, and **setup artifacts**. Each microservice is maintained in its own repository under [roboshopDevsecops](https://github.com/roboshopDevsecops).

## Source Code Repositories

| Component | Repository | Technology |
|-----------|------------|------------|
| Frontend | [roboshop-frontend](https://github.com/roboshopDevsecops/roboshop-frontend) | Next.js / Nginx |
| User Service | [roboshop-user](https://github.com/roboshopDevsecops/roboshop-user) | Node.js |
| Catalogue Service | [roboshop-catalogue](https://github.com/roboshopDevsecops/roboshop-catalogue) | Go |
| Cart Service | [roboshop-cart](https://github.com/roboshopDevsecops/roboshop-cart) | Node.js |
| Shipping Service | [roboshop-shipping](https://github.com/roboshopDevsecops/roboshop-shipping) | Java |
| Payment Service | [roboshop-payment](https://github.com/roboshopDevsecops/roboshop-payment) | Python |
| Ratings Service | [roboshop-ratings](https://github.com/roboshopDevsecops/roboshop-ratings) | Python |
| Orders Service | [roboshop-orders](https://github.com/roboshopDevsecops/roboshop-orders) | Java |
| Notification Service | Setup guide: [11-notification.md](11-notification.md) | Python |

> **Note** [roboshop-dispatch](https://github.com/roboshopDevsecops/roboshop-dispatch) is an additional supporting service in the organization.

## Deployment Guides

Step-by-step VM setup instructions (follow in order):

| Step | Guide | What it unlocks |
|------|-------|-----------------|
| 1 | [01-frontend.md](01-frontend.md) | Storefront UI (API calls fail until backends are up) |
| 2 | [02-mysql.md](02-mysql.md) | Shared relational database |
| 3 | [03-catalogue.md](03-catalogue.md) | Product listings |
| 4 | [04-mongodb.md](04-mongodb.md) | Document store for users |
| 5 | [05-user.md](05-user.md) | Registration and login |
| 6 | [06-valkey.md](06-valkey.md) | Cart session store |
| 7 | [07-cart.md](07-cart.md) | Add-to-cart |
| 8 | [08-shipping.md](08-shipping.md) | Shipping cost at checkout |
| 9 | [09-rabbitmq.md](09-rabbitmq.md) | Message broker |
| 10 | [10-payment.md](10-payment.md) | Checkout and payment |
| 11 | [11-notification.md](11-notification.md) | Order notifications |
| 12 | [12-orders.md](12-orders.md) | Order history |
| 13 | [13-ratings.md](13-ratings.md) | Product ratings |

## Platform & DevOps Repositories

| Purpose | Repository |
|---------|------------|
| VM-based Terraform (v1) | [roboshop-v1](https://github.com/roboshopDevsecops/roboshop-v1) |
| Kubernetes / EKS Terraform (v2) | [roboshop-v2](https://github.com/roboshopDevsecops/roboshop-v2) |
| Helm deployment charts | [roboshop-helm-v1](https://github.com/roboshopDevsecops/roboshop-helm-v1) |
| Reusable GitHub Actions | [github-reusable-workflows](https://github.com/roboshopDevsecops/github-reusable-workflows) |
| Grafana / Prometheus / AI observability | [ai-obs](https://github.com/roboshopDevsecops/ai-obs) |
| k6 / Locust load testing | [load-test](https://github.com/roboshopDevsecops/load-test) |

## Architecture

![RoboShop Microservices Architecture](architecture.jpg)

## Components

| Service      | Technology  | Port | Database            | Purpose                                            |
|--------------|-------------|------|---------------------|----------------------------------------------------|
| Frontend     | Next.js     | 3000 | —                   | Static React UI served by Nginx                    |
| User         | Node.js     | 8001 | MongoDB             | User registration, login, and profile management   |
| Catalogue    | Go          | 8002 | MySQL               | Product listings and category browsing             |
| Cart         | Node.js     | 8003 | Valkey               | Shopping cart session management with TTL          |
| Shipping     | Java        | 8004 | MySQL               | Shipping cost calculation and delivery options     |
| Payment      | Python      | 8005 | RabbitMQ            | Payment processing; publishes order events         |
| Ratings      | Python      | 8006 | MySQL               | Product ratings and reviews                        |
| Orders       | Java        | 8007 | MongoDB + RabbitMQ  | Order creation, persistence, and event consumption |
| Notification | Python      | 8008 | RabbitMQ            | Email/SMS notifications triggered by order events  |

## Server Allocation

> **Important** Record the **private IP** of every server after creation — each service configuration references the private IPs of its database and upstream dependencies.

| Server    | AWS       | Azure         | OS      | Runs                  |
|-----------|-----------|---------------|---------|-----------------------|
| Server 1  | t3.small  | Standard_B1ms  | RHEL 10 | Nginx + Frontend      |
| Server 2  | t3.small  | Standard_B1ms  | RHEL 10 | MySQL                 |
| Server 3  | t3.small  | Standard_B1ms  | RHEL 10 | Catalogue Service     |
| Server 4  | t3.small  | Standard_B1ms  | RHEL 10 | MongoDB               |
| Server 5  | t3.small  | Standard_B1ms  | RHEL 10 | User Service          |
| Server 6  | t3.small  | Standard_B1ms  | RHEL 10 | Valkey                 |
| Server 7  | t3.small  | Standard_B1ms  | RHEL 10 | Cart Service          |
| Server 8  | t3.small  | Standard_B1ms  | RHEL 10 | Shipping Service      |
| Server 9  | t3.small  | Standard_B1ms  | RHEL 10 | RabbitMQ              |
| Server 10 | t3.small  | Standard_B1ms  | RHEL 10 | Payment Service       |
| Server 11 | t3.small  | Standard_B1ms  | RHEL 10 | Notification Service  |
| Server 12 | t3.small  | Standard_B1ms  | RHEL 10 | Orders Service        |
| Server 13 | t3.small  | Standard_B1ms  | RHEL 10 | Ratings Service       |

## Prerequisites

Before setting up any component, disable SELinux and the firewall on all servers:

```shell
setenforce 0
systemctl stop firewalld
systemctl disable firewalld
```

> **Note** In production, configure SELinux policies and firewall rules properly instead of disabling them.

## Setup Order

The setup follows the **UI journey** — each step unlocks a new feature visible in the browser.

1. **Frontend / Nginx** — set up first; the storefront loads but all API calls fail (demonstrates the problem)
2. **MySQL** — shared relational database needed by Catalogue, Ratings, and Shipping
3. **Catalogue Service** — product listings and categories now appear in the UI
4. **MongoDB** — document store needed by User Service
5. **User Service** — registration and login now work in the UI
6. **Valkey** — in-memory store needed by Cart Service
7. **Cart Service** — add-to-cart functionality now works
8. **Shipping Service** — shipping cost calculation works at checkout
9. **RabbitMQ** — message broker needed by Payment, Orders, and Notification
10. **Payment Service** — checkout and payment flow now works
11. **Notification Service** — must be up before Orders since Orders calls it on order creation
12. **Orders Service** — order history and confirmation now appear after payment
13. **Ratings Service** — product star ratings now display and can be submitted
