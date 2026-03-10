# Jenkins Ansible Docker Deployment

## Project Overview
This project demonstrates a complete CI/CD pipeline that builds a static music dashboard
web application using Jenkins, Docker, Ansible and deploys it on a worker node via DockerHub.

---

## Architecture
```
GitHub → Jenkins → Docker Build → DockerHub → Ansible → Worker Node (Container:8085)
```

---

## Tech Stack
| Tool | Purpose |
|---|---|
| Jenkins | CI/CD Pipeline Automation |
| Ansible | Configuration Management & Deployment |
| Docker | Containerization |
| DockerHub | Docker Image Registry |
| Nginx | Web Server inside Container |
| GitHub | Source Code Repository |

---

## Infrastructure
| Node | Role |
|---|---|
| Master Node | Jenkins + Ansible installed |
| Worker Node | Runs Docker container (172.31.8.65) |

---

## Project Structure
```
jenkins-ansible-docker/
├── src/
│   └── index.html
├── Dockerfile
├── Jenkinsfile
├── playbook.yml
├── inventory
├── pom.xml
├── .gitignore
└── README.md
```

---

## Setup Guide

### Step 1: Setup Master Node
```bash
sudo apt update
sudo apt install docker.io openjdk-17-jdk ansible -y
sudo systemctl start docker
```
> For Jenkins installation refer to the official documentation:
> https://www.jenkins.io/doc/book/installing/linux/

---

### Step 2: Setup Worker Node
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
```

---

### Step 3: Ansible Setup
> Ansible is installed on Master Node. SSH key is configured between
> Master and Worker Node using ansible user for passwordless deployment.

---

### Step 4: Fix Jenkins Docker Permission
```bash
sudo usermod -aG docker jenkins
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart jenkins
```

---

### Step 5: Add DockerHub Credentials in Jenkins
```
Manage Jenkins → Credentials → System → Global credentials
→ Add Credentials
    Kind     : Username with password
    Username : rohinicc
    Password : <your_dockerhub_password>
    ID       : dockerhub-cred
→ Create
```

---

### Step 6: Create Jenkins Pipeline Job
```
New Item → Pipeline → OK
→ Pipeline script from SCM → Git
  → URL: https://github.com/rohinicc/jenkins-ansible-docker.git
  → Branch: main → Script Path: Jenkinsfile
→ Save → Build Now
```

---

### Step 7: Verify Deployment
```
http://172.31.8.65:8085
```

---

## Pipeline Stages
```
1. Clone Repository      → Pull code from GitHub
2. Docker Version Check  → Verify Docker installed
3. Build Docker Image    → Build from Dockerfile
4. Tag Docker Image      → Tag with BUILD_ID and latest
5. Push to DockerHub     → Push to rohinicc/music-dashboard-app
6. Validate Ansible      → Dry run playbook --check
7. Deploy via Ansible    → Deploy container on worker node
```

---

## Troubleshooting
| Error | Fix |
|---|---|
| `sudo: password required` | `echo "jenkins ALL=(ALL) NOPASSWD: ALL" \| sudo tee /etc/sudoers.d/jenkins` |
| `docker.sock permission denied` | `sudo usermod -aG docker jenkins && sudo chmod 666 /var/run/docker.sock` |
| `Dockerfile not found` | Rename `dockerfile` → `Dockerfile` (capital D) |
| `MissingPropertyException` | Use `DOCKER_USER` / `DOCKER_PASS` inside `withCredentials` |
| `ansible ping failed` | Run `ssh-copy-id ansible@172.31.8.65` on master |
| `port 8085 not accessible` | Open port 8085 in AWS Security Group inbound rules |
| `VERSION not expanding` | Use `"${BUILD_ID}"` not `'$BUILD_ID'` |

---

## Access Application
```
http://172.31.8.65:8085
```

---

## Author
- **GitHub:** rohinicc
- **DockerHub:** rohinicc
