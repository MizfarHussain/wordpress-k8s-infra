# WordPress Kubernetes Production Deployment (Helm + Minikube)

This project deploys a production-style WordPress application on
Kubernetes using Nginx (OpenResty with Lua), MySQL, Persistent Volumes,
and Helm for deployment. The setup also includes monitoring and alerting
using Prometheus and Grafana.

This implementation is designed for DevOps and Infrastructure internship
assignment practice, focusing on clean structure, reproducibility, and
production-oriented design.

## Overview

The deployment stack includes:

-   WordPress (PHP-FPM)
-   MySQL database
-   Nginx reverse proxy using OpenResty with Lua
-   Persistent Volumes (ReadWriteMany)
-   Kubernetes deployment via Helm chart
-   Prometheus and Grafana monitoring
-   Nginx request and 5xx failure metrics

## Prerequisites

The following tools must be installed:

-   Docker
-   kubectl
-   Minikube
-   Helm

Verify installation before continuing.

## Step 1: Start Minikube

Start Minikube and enable the metrics server. Configure Docker to build
images inside the Minikube environment.

    minikube start --memory=4096 --cpus=4
    minikube addons enable metrics-server
    eval $(minikube docker-env)

## Step 2: Build Docker Images

The project includes three custom images:

-   MySQL
-   WordPress (PHP-FPM)
-   Nginx (OpenResty + Lua metrics)

Build the images:

    docker build -t wp-mysql:1.0 docker/mysql
    docker build -t wp-app:1.0 docker/wordpress
    docker build -t wp-nginx:1.0 docker/nginx

These images will be used directly inside the Minikube cluster.

## Step 3: Create Persistent Volumes

Apply the Persistent Volume and Persistent Volume Claim definition:

    kubectl apply -f k8s/pv.yaml

This provides shared ReadWriteMany storage for MySQL and WordPress.

## Step 4: Deploy the Application Using Helm

Install the Helm release:

    helm install wp-release helm/wordpress

Verify the deployment:

    kubectl get pods,svc

Access WordPress via NodePort:

    minikube service nginx --url

Open the URL in a browser to complete the WordPress installation.

## Step 5: Deploy Monitoring Stack (Prometheus and Grafana)

Install the Kubernetes monitoring stack:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm install monitoring prometheus-community/kube-prometheus-stack

## Step 6: Enable Nginx Metrics Scraping

Apply the ServiceMonitor configuration:

    kubectl apply -f k8s/nginx-servicemonitor.yaml

This enables Prometheus to scrape metrics exposed by Lua inside Nginx.

## Step 7: Apply Alert Rules

Apply custom alerting rules for high CPU usage and Nginx 5xx failures:

    kubectl apply -f k8s/alerts.yaml

## Step 8: Access Grafana

Forward the Grafana service:

    kubectl port-forward svc/monitoring-grafana 3000:80

Open the dashboard:

    http://localhost:3000

Default credentials:

-   Username: admin
-   Password: prom-operator

Create visualizations for:

-   Pod CPU utilization
-   Total Nginx request count
-   Nginx 5xx error rate

## Cleanup

To remove all deployed resources:

    helm delete wp-release
    helm delete monitoring
    minikube delete

## Repository Structure

    wordpress-k8s-infra/
    ├── docker/
    │   ├── mysql/Dockerfile
    │   ├── wordpress/Dockerfile
    │   └── nginx/
    │       ├── Dockerfile
    │       └── nginx.conf
    ├── k8s/
    │   ├── pv.yaml
    │   ├── nginx-servicemonitor.yaml
    │   └── alerts.yaml
    ├── helm/
    │   └── wordpress/
    │       ├── Chart.yaml
    │       ├── values.yaml
    │       └── templates/
    │           ├── mysql.yaml
    │           ├── wordpress.yaml
    │           ├── nginx.yaml
    │           └── ingress.yaml (optional)
    └── README.md

## Notes

-   This setup is intended for learning and assessment environments.
-   Nginx exposes custom metrics using Lua for Prometheus.
-   All Docker images are built locally for Minikube use.
-   The deployment is fully reproducible and can be extended for cloud
    environments.
