# Jenkins Distributed Build Architecture on AWS EC2

A robust, containerized Jenkins distributed build environment running on an AWS EC2 instance using Docker and custom networks.

## Architecture Overview

- **Controller**: Runs inside a Docker container (`jenkins/jenkins:lts-jdk17`), exposing port `9090` for the web UI and port `50000` for inbound agent communication.
- **Agent**: Runs in a separate container (`jenkins/inbound-agent:latest`) connected via a shared Docker bridge network (`jenkins-net`).
- **Infrastructure**: Hosted on an AWS EC2 instance with port `50000` opened in the Security Group inbound rules to allow secure JNLP communication.

---

## Quick Setup Guide

### 1. Create a Custom Docker Network
```bash
docker network create jenkins-net
```

### 2. Run the Jenkins Controller
```bash
docker run -d \
  --name jenkins-controller \
  --network jenkins-net \
  -p 9090:8080 \
  -p 50000:50000 \
  -v ~/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts-jdk17
```

### 3. Run the Jenkins Inbound Agent
*(Make sure to generate your agent secret hash from your Jenkins dashboard under Manage Jenkins > Nodes)*
```bash
docker run -d \
  --name jenkins-agent1 \
  --network jenkins-net \
  --restart unless-stopped \
  -v ~/jenkins_agent_home:/home/jenkins/agent \
  jenkins/inbound-agent:latest \
  -url http://jenkins-controller:8080 \
  -secret YOUR_SECRET_HASH \
  -name agent1
```

---

## Verification Test Pipeline
To verify distributed build execution, create a Pipeline project in Jenkins restricted to the `linux-docker` node label with the following verification step:

```groovy
pipeline {
    agent {
        label 'linux-docker'
    }
    stages {
        stage('Verification') {
            steps {
                echo "Successfully running on agent node!"
                sh 'hostname'
                sh 'pwd'
                sh 'uname -a'
            }
        }
    }
}
```
