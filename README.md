# Simple Text Classification Model — AWS EKS Deployment

This project contains a basic machine learning text classification model deployed using Flask and containerized with Docker. The application is designed to be deployed on AWS Elastic Kubernetes Service (EKS).

Docker Hub image: [vimalpillai/mlops-aws-eks-deployment](https://hub.docker.com/r/vimalpillai/mlops-aws-eks-deployment)

## Project Overview

The application exposes a simple REST API that accepts text input and returns a classification prediction from the trained machine learning model.

## Tech Stack

- Python
- Machine Learning (scikit-learn)
- Flask
- Gunicorn
- Docker
- AWS EKS
- REST API

## Project Structure

```
.
├── app.py                   # Flask application (REST API)
├── wsgi.py                  # WSGI entrypoint
├── model/
│   ├── train.py             # Model training script (runs during docker build)
│   ├── intent_model.py      # Model wrapper used by the API
│   └── artifacts/           # Saved model artifacts
├── kubernetes-manifests/    # Kubernetes manifests for EKS deployment
├── Dockerfile
└── requirements.txt
```

## Run Locally (using Docker Hub image)

Follow the steps below to run the Dockerized application on your local machine.

### 1. Pull the Docker Image

The Docker image is available on Docker Hub: [vimalpillai/mlops-aws-eks-deployment](https://hub.docker.com/r/vimalpillai/mlops-aws-eks-deployment)

```bash
docker pull vimalpillai/mlops-aws-eks-deployment:v1
```

### 2. Run the Container

Start the application using:

```bash
docker run -d -p 4000:6000 vimalpillai/mlops-aws-eks-deployment:v1
```

This maps the local port `4000` to port `6000` inside the container.

Once the container is running, the API will be accessible at:

```
http://localhost:4000/
```

## Build the Image from Source

To build the Docker image yourself instead of pulling it from Docker Hub:

```bash
docker build -t vimalpillai/mlops-aws-eks-deployment:v1 .
```

The build performs the following steps:

1. Uses the official `python:3.10-slim` base image
2. Installs the Python dependencies from `requirements.txt`
3. Copies the application code and model files
4. Trains the model by running `model/train.py`
5. Exposes port `6000` and starts the app with Gunicorn (4 workers)

Optionally, push the built image to Docker Hub:

```bash
docker push vimalpillai/mlops-aws-eks-deployment:v1
```

### Run Locally (without Docker)

If you prefer to run the app directly with Python:

```bash
pip install -r requirements.txt
python model/train.py
python app.py
```

The API will be available at `http://localhost:6000/`.

## Prediction API

The model provides a `/predict` endpoint for making predictions.

### Endpoint

```
POST http://localhost:4000/predict
```

### Request

Send the input text as JSON:

```bash
curl -X POST http://localhost:4000/predict \
  -H "Content-Type: application/json" \
  -d '{"text":"not interested"}'
```

### Request Body

```json
{
  "text": "not interested"
}
```

The API will process the text using the machine learning model and return the corresponding prediction.

### Health Check

A `/health` endpoint is also available for health checks:

```
GET http://localhost:4000/health
```

## API Flow

```
Client
   |
   | POST /predict
   v
Flask API
   |
   v
ML Text Classification Model
   |
   v
Prediction
   |
   v
JSON Response
```

## AWS EKS Deployment

The same Dockerized application can be deployed to Amazon EKS (Elastic Kubernetes Service).

### Typical Deployment Flow

```
Machine Learning Model
        |
        v
Flask Application
        |
        v
Docker Image
        |
        v
Docker Hub
        |
        v
AWS EKS
        |
        v
Kubernetes Service
        |
        v
REST API
```

### Deploying to EKS

The Kubernetes manifests are provided in the `kubernetes-manifests/` directory:

```bash
# Update the image name in kubernetes-manifests/deployment.yaml
# image: vimalpillai/mlops-aws-eks-deployment:v1

kubectl apply -f kubernetes-manifests/namespace.yml
kubectl apply -f kubernetes-manifests/deployment.yaml
kubectl apply -f kubernetes-manifests/service.yaml
```

## Quick Start

```bash
# Pull the image
docker pull vimalpillai/mlops-aws-eks-deployment:v1

# Run the container
docker run -d -p 4000:6000 vimalpillai/mlops-aws-eks-deployment:v1

# Test the prediction API
curl -X POST http://localhost:4000/predict \
  -H "Content-Type: application/json" \
  -d '{"text":"not interested"}'
```
