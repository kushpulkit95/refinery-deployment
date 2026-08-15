# Refinery Deployment

![Docker](https://img.shields.io/badge/Docker-Containerization-blue)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red)
![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5)
![Helm](https://img.shields.io/badge/Helm-Package_Manager-0F1689)
![TCP%2FUDP](https://img.shields.io/badge/Networking-TCP%2FUDP-informational)
![Status](https://img.shields.io/badge/Status-Completed-success)

Deployment and infrastructure configuration for the **Data Refinery Engine Simulator** project.

This repository brings together the deployment configurations required to run the **Data Refinery Simulator** and **Receiver Server** across different environments.

The repository supports:

- Docker Compose deployment
- AWS EC2 deployment using Docker Compose
- Kubernetes deployment
- Minikube-based local Kubernetes validation
- Helm-based Kubernetes deployment and configuration

---

## Project Components

The overall Data Refinery system consists of two independent applications:

- **Data Refinery Simulator**
  - Generates CDR and NAT records
  - Sends CDR records over TCP
  - Sends NAT records over UDP
  - Performs finite simulation runs

- **Receiver Server**
  - Continuously listens for incoming records
  - Receives CDR over TCP
  - Receives NAT over UDP
  - Writes records to flat CSV files
  - Maintains runtime reception counters

The application repositories are maintained separately from this deployment repository.

---

## Architecture

```text
                         GitHub
                           │
                           ▼
                        Jenkins
                           │
                  Build + Docker Image
                           │
                           ▼
                      Docker Hub
                     /           \
                    /             \
                   ▼               ▼
             AWS EC2          Local PC
                   │               │
             Docker Compose     Minikube
                   │               │
                   │              Helm
                   │               │
                   ▼               ▼
              Simulator       Simulator
                   │               │
                   ▼               ▼
               Receiver        Receiver
                   │               │
                   └───────┬───────┘
                           ▼
                       CSV Output
```

### Deployment Environments

| Environment | Technology | Purpose |
|---|---|---|
| Local development | Java / Spring Boot | Application development and debugging |
| AWS EC2 | Docker + Docker Compose | Cloud deployment and container validation |
| Local Minikube | Kubernetes | Kubernetes orchestration validation |
| Local Minikube | Helm | Kubernetes packaging and configuration |

> **Note:** Kubernetes and Helm were validated locally using Minikube. They were not deployed to the AWS EC2 environment.

---

## Repository Structure

```text
refinery-deployment/
│
├── docker-compose.yml
│
├── k8s/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── receiver-deployment.yaml
│   ├── receiver-service.yaml
│   └── simulator-deployment.yaml
│
├── helm/
│   └── refinery/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── configmap.yaml
│           ├── receiver-deployment.yaml
│           ├── receiver-service.yaml
│           └── simulator-deployment.yaml
│
└── README.md
```

---

# Docker Compose

Docker Compose is used to run the Simulator and Receiver together as a multi-container application.

```text
┌─────────────────────────────┐
│       Docker Compose        │
│                             │
│  ┌────────────┐             │
│  │ Simulator  │             │
│  └─────┬──────┘             │
│        │                    │
│        │ TCP / UDP          │
│        ▼                    │
│  ┌────────────┐             │
│  │  Receiver  │             │
│  └─────┬──────┘             │
│        │                    │
│        ▼                    │
│     CSV files               │
└─────────────────────────────┘
```

The Simulator communicates with the Receiver using the Docker Compose service name rather than `localhost`.

```text
CDR ── TCP : 5000 ──► Receiver
NAT ── UDP : 5001 ──► Receiver
```

### Start the Deployment

```bash
docker compose up -d
```

### Check Running Containers

```bash
docker compose ps
```

### View Logs

```bash
docker compose logs
```

View a specific service:

```bash
docker compose logs simulator
docker compose logs receiver
```

### Stop the Deployment

```bash
docker compose down
```

---

# AWS EC2 Deployment

The containerized Data Refinery system was deployed and validated on an **AWS EC2 instance** using Docker Compose.

The AWS deployment flow is:

```text
GitHub
   │
   ▼
Jenkins
   │
   ▼
Docker Hub
   │
   ▼
AWS EC2
   │
   ▼
Docker Compose
   │
   ├── Simulator
   │
   └── Receiver
```

The AWS environment was used to validate:

- Docker image retrieval from Docker Hub
- Simulator startup
- Receiver startup
- TCP communication
- UDP communication
- Record transmission
- Receiver-side CSV output

AWS EC2 was used for the Docker/Compose deployment. Kubernetes and Helm were subsequently validated separately using local Minikube.

---

# CI/CD Pipeline

The application repositories are integrated with Jenkins for automated build and container image publishing.

The general workflow is:

```text
GitHub
   │
   ▼
Jenkins
   │
   ├── Checkout
   ├── Maven Build
   ├── Docker Image Build
   ├── Docker Image Tag
   └── Docker Image Push
           │
           ▼
       Docker Hub
```

The primary images are:

```text
pkistrying/data-refinery-simulator:latest
pkistrying/receiver-server:latest
```

---

# Kubernetes Deployment

Kubernetes deployment was developed and validated locally using **Minikube**.

The Kubernetes environment contains:

```text
Namespace
   │
   ├── Receiver Deployment
   │      │
   │      └── Receiver Pod
   │
   ├── Receiver Service
   │      ├── 5000/TCP
   │      └── 5001/UDP
   │
   └── Simulator Deployment
          │
          └── Simulator Pod
```

---

# Kubernetes Namespace

The project uses the namespace:

```text
refinery
```

Create the namespace:

```bash
kubectl create namespace refinery
```

Check the namespace:

```bash
kubectl get namespace refinery
```

---

# Kubernetes Configuration

The Simulator receives runtime configuration through a Kubernetes ConfigMap.

The configuration includes:

```text
RECORDCOUNT
DATATYPE
TIMESTAMP
TIMEPERIOD

CDRHOST
CDRPORT

NATHOST
NATPORT
```

The Receiver Service name is:

```text
refinery-receiver-service
```

Therefore:

```text
CDRHOST = refinery-receiver-service
CDRPORT = 5000

NATHOST = refinery-receiver-service
NATPORT = 5001
```

This avoids using `localhost` inside the Kubernetes environment.

---

# Kubernetes Deployment Commands

Apply the Kubernetes manifests:

```bash
kubectl apply -f k8s/ -n refinery
```

Check all resources:

```bash
kubectl get all -n refinery
```

Check Pods:

```bash
kubectl get pods -n refinery
```

Check Services:

```bash
kubectl get services -n refinery
```

Check Deployments:

```bash
kubectl get deployments -n refinery
```

---

# Kubernetes Logs

View Simulator logs:

```bash
kubectl logs deployment/refinery-simulator -n refinery
```

View Receiver logs:

```bash
kubectl logs deployment/refinery-receiver -n refinery
```

Follow Receiver logs continuously:

```bash
kubectl logs -f deployment/refinery-receiver -n refinery
```

---

# Kubernetes Inspection

Describe a Pod:

```bash
kubectl describe pod <pod-name> -n refinery
```

Open a shell inside the Simulator:

```bash
kubectl exec -it deployment/refinery-simulator -n refinery -- sh
```

Open a shell inside the Receiver:

```bash
kubectl exec -it deployment/refinery-receiver -n refinery -- sh
```

Check Simulator environment variables:

```bash
kubectl exec deployment/refinery-simulator -n refinery -- sh -c "echo RECORDCOUNT=$RECORDCOUNT && echo DATATYPE=$DATATYPE && echo TIMEPERIOD=$TIMEPERIOD"
```

Check the Receiver Service endpoints:

```bash
kubectl get endpoints refinery-receiver-service -n refinery
```

For newer Kubernetes versions:

```bash
kubectl get endpointslice -n refinery
```

---

# Receiver Service

The Receiver is exposed internally through:

```text
refinery-receiver-service
```

Ports:

| Port | Protocol | Purpose |
|---:|---|---|
| 5000 | TCP | CDR |
| 5001 | UDP | NAT |

The Simulator sends:

```text
CDR
 │
 └── TCP : 5000
          │
          ▼
refinery-receiver-service
          │
          ▼
Receiver

NAT
 │
 └── UDP : 5001
          │
          ▼
refinery-receiver-service
          │
          ▼
Receiver
```

The Service uses `ClusterIP` because the Simulator and Receiver only need internal cluster communication.

---

# Helm

The Kubernetes resources are packaged as a Helm chart.

Chart structure:

```text
helm/
└── refinery/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── configmap.yaml
        ├── receiver-deployment.yaml
        ├── receiver-service.yaml
        └── simulator-deployment.yaml
```

Helm allows deployment configuration to be parameterized without manually editing every Kubernetes manifest.

---

# Helm Validation

Validate the chart:

```bash
helm lint helm/refinery
```

Render the Kubernetes manifests:

```bash
helm template refinery helm/refinery -n refinery
```

---

# Helm Installation

Install the chart:

```bash
helm install refinery helm/refinery -n refinery
```

Check installed releases:

```bash
helm list -n refinery
```

Check release status:

```bash
helm status refinery -n refinery
```

---

# Helm Upgrade

After changing:

```text
helm/refinery/values.yaml
```

upgrade the release:

```bash
helm upgrade refinery helm/refinery -n refinery
```

Check resources:

```bash
kubectl get all -n refinery
```

If Simulator configuration is supplied through environment variables from a ConfigMap, restart the Simulator:

```bash
kubectl rollout restart deployment/refinery-simulator -n refinery
```

Check rollout status:

```bash
kubectl rollout status deployment/refinery-simulator -n refinery
```

---

# Helm Uninstallation

Remove the Helm release:

```bash
helm uninstall refinery -n refinery
```

Verify:

```bash
kubectl get all -n refinery
```

---

# Helm Configuration

The primary Helm configuration is maintained in:

```text
helm/refinery/values.yaml
```

The configuration controls deployment-specific values such as:

- Simulator image
- Receiver image
- Replica counts
- Resource requests and limits
- Simulator record count
- Data type
- Timestamp
- Time period
- Receiver ports

Configuration flow:

```text
values.yaml
     │
     ▼
Helm Templates
     │
     ▼
Kubernetes Manifests
     │
     ├── ConfigMap
     ├── Deployments
     └── Service
```

---

# Application Images

The deployment uses Docker images published to Docker Hub.

### Simulator

```text
pkistrying/data-refinery-simulator:latest
```

### Receiver

```text
pkistrying/receiver-server:latest
```

The Kubernetes deployments use:

```yaml
imagePullPolicy: Always
```

for the configured `latest` images.

---

# Testing and Validation

The deployment repository was used to validate the complete system across Docker and Kubernetes environments.

Testing included different:

- `recordCount` values
- `datatype` values
- `timeperiod` values

The following were validated:

- Simulator record generation
- CDR TCP transmission
- NAT UDP transmission
- Simulator sent count
- Receiver received count
- CDR CSV output
- NAT CSV output
- Multiple Simulator executions
- Continuous Receiver operation
- Kubernetes Service communication
- ConfigMap-based configuration
- Helm installation
- Helm upgrade
- Helm uninstallation

---

# End-to-End Test Flow

```text
Configure Simulator
        │
        ▼
Deploy Receiver
        │
        ▼
Deploy Simulator
        │
        ▼
Run Simulator
        │
        ├───────────────┐
        ▼               ▼
   CDR / TCP        NAT / UDP
        │               │
        └───────┬───────┘
                ▼
            Receiver
                │
                ▼
          Flat CSV files
                │
                ▼
       Validate test results
```

---

# Test Validation

For a configuration such as:

```text
recordCount = 5
datatype    = cdr,nat
```

the expected result is:

```text
CDR:
Generated = 5
Sent      = 5
Received  = 5

NAT:
Generated = 5
Sent      = 5
Received  = 5
```

For each output CSV:

```text
1 header + 5 records = 6 lines
```

When the Simulator is executed again without restarting the Receiver, the Receiver's cumulative counters should increase accordingly.

Example:

```text
First run:
CDR Total = 5
NAT Total = 5

Second run:
CDR Total = 10
NAT Total = 10
```

This validates that the Receiver operates continuously rather than requiring a restart for every simulation batch.

---

# Troubleshooting

## Pod is in CrashLoopBackOff

Check logs:

```bash
kubectl logs deployment/refinery-receiver -n refinery
```

or:

```bash
kubectl logs deployment/refinery-simulator -n refinery
```

Describe the Pod:

```bash
kubectl describe pod <pod-name> -n refinery
```

---

## Simulator cannot reach Receiver

Open a shell:

```bash
kubectl exec -it deployment/refinery-simulator -n refinery -- sh
```

Check:

```bash
echo $CDRHOST
echo $CDRPORT
echo $NATHOST
echo $NATPORT
```

Expected:

```text
CDRHOST=refinery-receiver-service
CDRPORT=5000

NATHOST=refinery-receiver-service
NATPORT=5001
```

Check the Service:

```bash
kubectl get service refinery-receiver-service -n refinery
```

Check endpoints:

```bash
kubectl get endpoints refinery-receiver-service -n refinery
```

---

## Configuration Changes Are Not Reflected

After changing Helm values:

```bash
helm upgrade refinery helm/refinery -n refinery
```

Restart the Simulator:

```bash
kubectl rollout restart deployment/refinery-simulator -n refinery
```

Verify:

```bash
kubectl exec deployment/refinery-simulator -n refinery -- sh -c "echo RECORDCOUNT=$RECORDCOUNT && echo DATATYPE=$DATATYPE && echo TIMEPERIOD=$TIMEPERIOD"
```

---

## New Docker Image Is Not Being Used

Check the Deployment:

```bash
kubectl describe deployment refinery-simulator -n refinery
```

Restart if necessary:

```bash
kubectl rollout restart deployment/refinery-simulator -n refinery
```

Verify:

```bash
kubectl get pods -n refinery
```

---

# Useful Command Reference

## Docker

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

## Kubernetes

```bash
kubectl get all -n refinery
kubectl get pods -n refinery
kubectl get services -n refinery
kubectl logs deployment/refinery-simulator -n refinery
kubectl logs deployment/refinery-receiver -n refinery
kubectl describe pod <pod-name> -n refinery
kubectl exec -it <pod-name> -n refinery -- sh
kubectl rollout restart deployment/refinery-simulator -n refinery
kubectl rollout restart deployment/refinery-receiver -n refinery
```

## Helm

```bash
helm lint helm/refinery
helm template refinery helm/refinery -n refinery
helm install refinery helm/refinery -n refinery
helm list -n refinery
helm status refinery -n refinery
helm upgrade refinery helm/refinery -n refinery
helm uninstall refinery -n refinery
```

---

# Deployment Lifecycle

```text
                   SOURCE CODE
                       │
                       ▼
                    GitHub
                       │
                       ▼
                    Jenkins
                       │
              ┌────────┴────────┐
              │                 │
         Maven Build       Docker Build
              │                 │
              └────────┬────────┘
                       ▼
                   Docker Hub
                   /         \
                  /           \
                 ▼             ▼
             AWS EC2       Local Minikube
                 │             │
          Docker Compose       │
                 │            Helm
                 │             │
                 ▼             ▼
             Simulator     Simulator
                 │             │
                 ▼             ▼
              Receiver      Receiver
                 │             │
                 └──────┬──────┘
                        ▼
                    CSV Output
                        │
                        ▼
                 Testing Evidence
```

---

# Project Status

The deployment and infrastructure implementation has been completed and validated.

Completed:

- [x] Docker Compose deployment
- [x] AWS EC2 cloud deployment
- [x] Jenkins CI/CD integration
- [x] Docker Hub image publishing
- [x] Kubernetes manifests
- [x] Minikube Kubernetes deployment
- [x] Kubernetes ConfigMap
- [x] Kubernetes Service
- [x] Kubernetes resource configuration
- [x] Helm chart
- [x] Helm lint validation
- [x] Helm template validation
- [x] Helm install
- [x] Helm upgrade
- [x] Helm uninstall
- [x] Simulator configuration through Helm
- [x] End-to-end testing
- [x] Testing evidence collection

---

# Related Repositories

### Data Refinery Simulator

The Simulator generates CDR and NAT records and transmits them to the Receiver.

https://github.com/kushpulkit95/data-refinery-simulator

### Receiver Server

The Receiver continuously accepts CDR and NAT records and persists them into flat files.

https://github.com/kushpulkit95/receiver-server

---

# Author

**Pulkit Kush**

B.Tech Computer Science — Data Science

Internship Project

Java • Spring Boot • TCP/UDP • Docker • Jenkins • AWS EC2 • Kubernetes • Helm • CI/CD
