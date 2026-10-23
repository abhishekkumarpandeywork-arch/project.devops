# Linux Server Administration and Bash Automation

A hands-on DevOps project that automates the setup of an Ubuntu Linux server, deploys a sample LAMP application, schedules routine maintenance tasks, and monitors server health using Prometheus, Node Exporter, and Grafana.

## Project Overview

This project demonstrates core Linux administration and DevOps practices:

- Automated installation and configuration of a LAMP stack.
- Deployment of a sample PHP application connected to MySQL.
- Bash scripts for application/database backups, log cleanup, and health checks.
- Cron jobs to schedule routine maintenance.
- Prometheus and Node Exporter for collecting and scraping server metrics.
- Grafana for visualizing monitoring metrics.
- Basic service checks to validate the installation.

> **Environment:** Ubuntu Server 22.04/24.04 (as targeted by the setup script).

## Architecture

```text
                  Users / Browser
                         |
                         v
                 Apache Web Server
                         |
                         v
                    PHP App
                         |
                         v
                    MySQL DB

 Ubuntu Server Metrics
          |
          v
    Node Exporter (:9100)
          |
          v
      Prometheus (:9090)
          |
          v
       Grafana (:3000)

 Cron Scheduler
    |          |             |
    v          v             v
  Backup    Log Cleanup   Health Check
```

## Technologies Used

| Technology | Purpose |
|---|---|
| Ubuntu Linux | Server operating system |
| Bash | Administration and automation scripts |
| Apache | Web server |
| PHP | Sample web application |
| MySQL | Relational database |
| Cron | Scheduled automation |
| Prometheus | Metrics collection and monitoring |
| Node Exporter | Exposes Linux host metrics |
| Grafana | Monitoring dashboards |
| Git/GitHub | Version control and project documentation |

## Repository Contents

```text
.
├── README.md
└── setup_linux_project.sh
```

The setup script creates the application, supporting automation scripts, directories, service configuration, and scheduled jobs on the target Ubuntu server.

## Prerequisites

- An Ubuntu Server 22.04 or 24.04 machine or VM.
- A user account with `sudo` privileges.
- Internet connectivity for package installation.
- Sufficient disk space for packages, logs, and backups.
- Access to the required ports in the host firewall or cloud security group.

## Installation and Setup

### 1. Clone the repository

Clone your repository using the HTTPS URL below (or use the SSH URL if your GitHub SSH key is configured).

```bash
git clone https://github.com/abhishekkumarpandeywork-arch/project.devops.git
cd project.devops
```

If you downloaded the project ZIP instead, extract it and open a terminal in the extracted folder.

### 2. Make the script executable

```bash
chmod +x setup_linux_project.sh
```

### 3. Run the installation script

```bash
sudo bash setup_linux_project.sh
```

The script updates Ubuntu packages and installs/configures Apache, PHP, MySQL, Prometheus, Node Exporter, and Grafana. It also creates the sample PHP application, automation scripts, Cron jobs, and an initial backup.

Review the script before running it on any important server. It changes system packages and configuration, creates a database account, and modifies the root user's Cron schedule.

### 4. Access the services

The script prints the server IP address at the end.

| Service | URL |
|---|---|
| Sample PHP application | `http://SERVER_IP/` |
| Prometheus | `http://SERVER_IP:9090` |
| Grafana | `http://SERVER_IP:3000` |
| Node Exporter metrics | `http://SERVER_IP:9100/metrics` |

Replace `SERVER_IP` with your Ubuntu server's reachable IP address.

For an AWS EC2 deployment, configure the instance security group and host firewall as needed. Restrict monitoring ports to trusted IPs; do not expose them publicly without appropriate controls.

## Bash Automation

The setup script creates the following scripts under `/opt/linux-server-project/scripts/` and links them into `/usr/local/bin/`.

### 1. Application and database backup — `backup.sh`

- Creates a timestamped compressed archive of `/var/www/html`.
- Exports the `lampdb` MySQL database using `mysqldump`.
- Removes backup files older than seven days.
- Writes status output when run through the configured Cron job.

Backup location:

```text
/opt/linux-server-project/backups/
```

Run manually:

```bash
sudo backup.sh
```

### 2. Log cleanup — `log_cleanup.sh`

- Searches `/var/log` for files ending in `.log`.
- Deletes matching files older than seven days.

Run manually:

```bash
sudo log_cleanup.sh
```

**Caution:** Deleting system log files can affect troubleshooting and log rotation. Review this script and test it carefully before using it on a production system.

### 3. Server health check — `health_check.sh`

Records information about:

- System uptime and load
- Memory usage
- Root filesystem usage
- Apache and MySQL service status
- Node Exporter, Prometheus, and Grafana service status

Health-check log:

```text
/opt/linux-server-project/logs/health.log
```

Run manually:

```bash
sudo health_check.sh
```

## Scheduled Tasks

The setup script configures these Cron schedules:

| Schedule | Task |
|---|---|
| Daily at 02:00 | Back up web files and MySQL database |
| Daily at 03:00 | Clean up matching old `.log` files |
| Every hour | Run the server health check |

Check the configured root Cron jobs:

```bash
sudo crontab -l
```

Check generated logs:

```bash
sudo ls -lh /opt/linux-server-project/logs/
sudo tail -n 50 /opt/linux-server-project/logs/health.log
```

## Monitoring Setup

### Prometheus and Node Exporter

Node Exporter exposes Linux host metrics on port `9100`. Prometheus is configured to scrape the local Node Exporter target every 15 seconds.

Prometheus configuration:

```text
/etc/prometheus/prometheus.yml
```

Open Prometheus and check **Status → Targets**. The `ubuntu-server` target should be `UP` when the service is reachable and configured correctly.

### Grafana

1. Open `http://SERVER_IP:3000`.
2. Sign in and complete the initial password-change process if prompted.
3. Go to **Connections → Data sources → Add data source**.
4. Select **Prometheus**.
5. Configure the URL as `http://localhost:9090` when Grafana and Prometheus are on the same machine.
6. Save and test the data source.
7. Create or import a suitable Node Exporter dashboard to visualize CPU, memory, filesystem, and other available host metrics.

Do not assume a dashboard is already configured: add the data source and dashboard as part of the monitoring setup.

## Validation and Troubleshooting

Check service status:

```bash
sudo systemctl status apache2
sudo systemctl status mysql
sudo systemctl status prometheus-node-exporter
sudo systemctl status prometheus
sudo systemctl status grafana-server
```

Check listening ports:

```bash
sudo ss -tulpn
```

Test local endpoints:

```bash
curl -I http://localhost/
curl -s http://localhost:9090/-/healthy
curl -s http://localhost:9100/metrics | head
```

View service logs when a component fails:

```bash
sudo journalctl -u apache2 --no-pager -n 50
sudo journalctl -u mysql --no-pager -n 50
sudo journalctl -u prometheus --no-pager -n 50
sudo journalctl -u grafana-server --no-pager -n 50
```

If a URL does not load remotely, check the service status, server IP, firewall rules, cloud security-group inbound rules, and whether the service is listening on the expected port.

## Security Notes

This repository is intended for a learning lab, not a production-ready deployment.

- The current setup script contains a sample MySQL password (`Password@123`). **Change it before use**, and avoid committing real credentials to Git.
- The sample PHP application also contains the database credentials; update the application and database configuration together if you change them.
- Restrict ports `3000`, `9090`, and `9100` to trusted networks. Expose only the services that need public access.
- Review the log-cleanup behavior before running it on a real system.
- Use least-privilege accounts, secure credentials, backups, and appropriate patching for production.
- The script performs system updates and changes service, database, and Cron configuration. Test in a disposable VM first.

## Learning Outcomes

By completing this project, you can practice:

- Linux service and package administration.
- Bash scripting and command-line automation.
- LAMP stack deployment and basic database connectivity.
- Cron scheduling and log management.
- Backup creation and retention.
- Host metrics collection and monitoring.
- Troubleshooting with `systemctl`, `journalctl`, `curl`, and `ss`.
- Documenting an infrastructure project for GitHub.

## Future Improvements

- Move credentials into a secure configuration mechanism instead of hardcoding them.
- Add backup verification and a tested restore procedure.
- Use safer log rotation and a dry-run mode for cleanup.
- Add a custom Grafana dashboard and alerting rules.
- Add automated tests and a CI workflow for shell-script linting.
- Manage configuration with Ansible or provision infrastructure with Terraform.
- Add a reverse proxy and HTTPS for a more realistic deployment.

## Author

**Abhishek Kumar Pandey**  
DevOps / Linux Administration Project  
GitHub: [abhishekkumarpandeywork-arch](https://github.com/abhishekkumarpandeywork-arch)  
Repository: [project.devops](https://github.com/abhishekkumarpandeywork-arch/project.devops)
