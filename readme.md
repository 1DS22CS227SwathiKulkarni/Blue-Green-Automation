# Kaffeine Blue:Green - CI/CD Automation

This project showcases a fully automated CI/CD pipeline for the **Kaffeine Table Booking Application**, utilizing a **Blue-Green Deployment strategy** to ensure **zero-downtime updates** and **instant rollback capability**.

## Tech Stack

- **Backend**: Flask (Python)
- **Database**: MongoDB Atlas
- **DevOps Tools**: 
  - GitHub (Source control + Webhooks)
  - Jenkins (CI/CD automation)
  - Docker & DockerHub (Containerization)
  - Kubernetes via Minikube (Orchestration)
  - Ngrok (Port tunneling for Jenkins)
- **Environment**: Windows OS, PowerShell
- **IDE**: VS Code

## Key Features

- Blue-Green Deployment strategy for zero-downtime.
- Real-time CI/CD pipeline triggered on GitHub commits.
- Dockerized Flask app hosted on Kubernetes using Minikube.
- Persistent storage with MongoDB Atlas.
- Admin panel for managing café bookings.
- Real-time table booking UI for users.

## Problem Statement

Frequent feature updates in web applications demand a reliable deployment strategy. Manual processes can lead to errors, downtime, and slow rollbacks. This project introduces DevOps practices to streamline and automate deployments with minimal human intervention, ensuring high availability and data integrity.

## Proposed Solution

- Code commit → GitHub webhook triggers Jenkins
- Jenkins:
  - Pulls latest code
  - Builds Docker image
  - Pushes to DockerHub
- Kubernetes (Minikube):
  - Deploys Blue environment
  - Switches traffic to Green environment after validation
  - Maintains rollback capability
- MongoDB Atlas ensures user data persistence throughout.

## Application Screenshots

- Home Page
<img width="772" height="458" alt="image" src="https://github.com/user-attachments/assets/86208de5-6f6e-418b-b33d-0bb7ab46ad0a" />

- Jenkins Build Status
<img width="772" height="413" alt="image" src="https://github.com/user-attachments/assets/996c812e-686c-4fe8-b424-f1a459b95917" />
  
- Minikube Terminal Status
<img width="772" height="194" alt="image" src="https://github.com/user-attachments/assets/58ac7aa6-e0b5-4665-a1c5-c1b594a516fc" />

- WebHooks Snapshots
<img width="772" height="229" alt="image" src="https://github.com/user-attachments/assets/9a189c8f-9e5b-4592-905f-9ad3a59fa0c6" />

- Blue/Green Deployment Toggle UI
<img width="772" height="456" alt="image" src="https://github.com/user-attachments/assets/61faf7db-eb6a-4533-ba0d-4c2c688763a2" />

## Project Structure

- **Blue-Green/**
  - **ADMIN/**
    - static/
    - templates/
    - app2.py
    - booktemp.html
    - calculate.py
    - password.text
    - nginx.conf
  - **User/**
    - __pycache__/
    - **k8s/**
      - nginx.conf
      - blue-deployment-template.yaml
      - blue-deployment.yaml
      - green-deployment.yaml
      - service.yaml
    - static/
    - templates/
    - .dockerignore
    - app.py
    - booktemp.html
    - calculate.py
    - Dockerfile
    - nginx.conf
    - password.text
    - requirements.txt
  - .gitignore
  - Jenkinsfile
  - readme.md
  - temp/

> This project demonstrates real-world DevOps implementation for continuous delivery and deployment automation with Blue-Green strategy using modern cloud-native tools.
