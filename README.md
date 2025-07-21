# EC2 Monitoring with Prometheus, Node Exporter, and Grafana using Terraform + Ansible

## Project Overview

This project sets up a basic monitoring stack on AWS using **Terraform** for infrastructure provisioning and **Ansible** for configuration management. It installs **Prometheus** on a `monitoring_server`, and **Grafana** + **Node Exporter** on an `app_server`, enabling system-level metric collection and visualization.

---

## Tools and Services Used

- **Terraform** – Infrastructure as Code (IaC)
- **Ansible** – Configuration management
- **Prometheus** – Metrics collection
- **Grafana** – Metrics visualization
- **Node Exporter** – Exposes system metrics
- **AWS EC2** – Virtual servers
- **Ubuntu** – Operating system for instances

---

## Prerequisites

- AWS CLI configured
- Terraform installed
- WSL (Windows Subsystem for Linux) and Ubuntu installed 
- Ansible installed on your local machine
- Basic knowledge of AWS, SSH, and Linux
- Public-private keypair (used to SSH into EC2)

---

## Features

- Launches two EC2 instances using Terraform
- Automatically installs Prometheus, Grafana, and Node Exporter using Ansible
- Exposes system metrics of app server
- Dashboards system metrics in Grafana via Node Exporter
- No custom AMI dependency — installs packages live

---

## How It Works

1. **Terraform** provisions the infrastructure (monitoring and app servers).
2. **Ansible** installs:
   - On **monitoring_server**: Prometheus 
   - On **app_server**: Grafana + Node Exporter
3. **Prometheus** scrapes system metrics from app server.
4. **Grafana** visualizes the data from Prometheus.

---

## Architecture Diagram

<img width="1087" height="704" alt="Monitoring-diagram" src="https://github.com/user-attachments/assets/46a5b573-b6ae-47ce-8515-a3fb55a03688" />

---

## Project Structure
```bash
aws-monitoring-automation/
├── terraform/
│ ├── main.tf
│ ├── variables.tf
│ └── outputs.tf
│ └── terraform.tfvars
├── ansible/
│ ├── playbook.yml
│ ├── inventory.ini
│ └── roles/
│ ├── prometheus/tasks/main.yml
│ ├── grafana/tasks/main.yml
│ └── node_exporter/tasks/main.yml
├── .gitignore
└── README.md
```

---

## Steps to Run

### 1. Clone the repo initialize infrastructure

```bash
git clone https://github.com/Shravani3001/aws-monitoring-automation.git
cd aws-monitoring-automation/terraform
```

### 2. SSH key pair generated using:
```bash
ssh-keygen -t rsa -b 4096 -f monitoring-key
```

### 3. Initialize infrastructure
```bash
terraform init
terraform apply
```

### 4. Install Ansible

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ansible -y
```

**macOS (Homebrew):**
```bash
brew install ansible
```

**Windows (WSL recommended):**

Install WSL + Ubuntu

Inside WSL terminal:
```bash
sudo apt update
sudo apt install ansible -y
```

**Verify:**
```bash
ansible --version
```

### 5. Navigate to the Project Folder

To run Ansible and move the SSH key, you need to be inside your project folder from within WSL.

If your project is saved somewhere on your Windows file system (like in C: or D:), access it through /mnt/. For example:
```bash
cd /mnt/c/Users/your-windows-username/path-to-your-project
cd /mnt/c/Users/your-username/aws-monitoring-automation
```

Replace:

- c with your actual Windows drive letter (e.g., d if the project is in D drive).
- your-windows-username with your actual Windows username.
- path-to-your-project with the folder path where you cloned/downloaded the project.

### 6. Setup SSH key

Inside Ubuntu:
```bash
mkdir -p ~/ssh-keys
cp terraform/monitoring-key ~/ssh-keys/
chmod 400 ~/ssh-keys/monitoring-key
ls -l ~/ssh-keys/monitoring-key   # Confirm permissions: -r--------
```
- Creates a folder called ssh-keys inside your Linux user’s home directory — where we’ll safely store the SSH key file. 
- Copies your private key into the ~/ssh-keys/ folder inside WSL Linux (so we avoid the Windows permission problem).
- Makes the key secure, so SSH/Ansible won’t reject it.
 Only you can read the file, no one else can access it.


### 7. Update inventory and playbook

- Replace the public IPs in ansible/inventory.ini
- Update Prometheus config in prometheus.yml to point to both localhost:9100 and app server private IP

### 8. Run Ansible Playbook
```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```

### 8. Access Prometheus

Port-forward to your local machine:
```bash
cd ../terraform/
ssh -i ~/ssh-keys/monitoring-key -L 9090:localhost:9090 ubuntu@monitoring-server-public-ip
```

Then open:
```bash
http://localhost:9090/targets

http://localhost:9090
```

**Try a query like:**
```bash
rate(node_cpu_seconds_total{mode="user"}[1m])
```

This query gives you the rate of CPU usage in user mode over the last 1 minute for all CPUs on your monitored nodes.

<img width="1920" height="959" alt="Output 1" src="https://github.com/user-attachments/assets/81bc6ce5-6609-4b19-b2cf-d20669d30576" />


<img width="1920" height="907" alt="Output 2" src="https://github.com/user-attachments/assets/c2e96c22-b95e-4681-8732-fa52e9da6202" />


### 9. Verify Node Exporter

SSH into each server and run:
```bash
curl http://localhost:9100/metrics
```

### 10. Access Grafana Dashboard

Port-forward:

Go back to the terraform folder and run 

```bash
ssh -i ~/ssh-keys/monitoring-key -L 3000:localhost:3000 ubuntu@app-server-public-ip
```

Then go to:
```bash
http://localhost:3000
```

**Login:**

Username: admin

Password: admin

<img width="1920" height="961" alt="Output 3" src="https://github.com/user-attachments/assets/d36da64f-c83d-4408-ab67-7bdcbf1d32fc" />


### 11. Configure Prometheus as Data Source

- Go to Grafana → Configuration → Data Sources
- Add Prometheus
- URL: http://monitoring-server-private-ip:9090
- Save & Test

<img width="1920" height="959" alt="Output 4" src="https://github.com/user-attachments/assets/7b5b9387-100b-4733-bd4a-41f9bb5be5e1" />

### 12. Import Dashboard

- Go to Dashboards → New → Import
- Paste ID: 1860 (Node Exporter Full)
- Select the Prometheus data source 
- In the dashboard, under instance, select localhost:9100 to view metrics

<img width="1920" height="959" alt="Output 5" src="https://github.com/user-attachments/assets/303e2c5d-789c-4e71-ba02-6b6c1b3ef724" />



✅ Output

Prometheus shows both localhost:9100 and app-private-ip:9100 as UP under /targets

**Grafana dashboard visualizes:**
- CPU usage
- Memory usage
- Disk I/O
- Network stats

## About Me

I'm Shravani, a self-taught and project-driven DevOps engineer passionate about building scalable infrastructure and automating complex workflows.

I love solving real-world problems with tools like Terraform, Ansible, Docker, Jenkins, and AWS — and I’m always learning something new to sharpen my edge in DevOps.

**Connect with me:**

[LinkedIn](https://www.linkedin.com/in/shravani3001) 

[GitHub](https://github.com/Shravani3001)
