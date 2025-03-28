
# microservice_based_app_project

---

# Multi-Stack Voting Application

**Welcome to DevOps practice project!** This repository hosts a multi-stack voting application composed of several services, each implemented in a different language and technology stack. The goal is to gain experience with containerization, orchestration, and running a distributed set of services—both individually and as part of a unified system in aws infrastructure to ensure communication within frontend and backend as well as database.

This application, while simple, uses multiple components commonly found in modern distributed architectures, provides hands-on practice in connecting services, handling containers, and working with basic infrastructure automation.

## Application Overview

![Architecture](/image.png)

The voting application includes:

- **Vote (Python)**: A Python Flask-based web application where users can vote between two options.
- **Redis (in-memory queue)**: Collects incoming votes and temporarily stores them.
- **Worker (.NET)**: A .NET 7.0-based service that consumes votes from Redis and persists them into a database.
- **Postgres (Database)**: Stores votes for long-term persistence.
- **Result (Node.js)**: A Node.js/Express web application that displays the vote counts in real time.

### Why This Setup?

The goal is to integrate a variety of languages, tools, and frameworks in one place. This is **not** a perfect production design. Instead, it’s intentionally diverse for knowledge/practice:

- Work with multiple runtimes and languages (Python, Node.js, .NET).
- Interact with services like Redis and Postgres.
- Containerize applications using Docker.
- Use Docker Compose to orchestrate and manage multiple services together.

By dealing with this “messy” environment, you’ll build real-world problem-solving skills. After this project, feel more confident tackling more complex deployments and troubleshooting issues in containerized, multi-service setups.

---

## How to Run Each Component

### Running the Vote Service (Python) Locally (No Docker)

1. Ensure you have Python 3.10+ installed.
2. Navigate to the `vote` directory:
   ```bash
   cd vote
   pip install -r requirements.txt
   python app.py
   ```
   Access the vote interface at [http://localhost:5000](http://localhost:5000).

### Running Redis Locally (No Docker)

1. Install Redis on your system ([https://redis.io/docs/getting-started/](https://redis.io/docs/getting-started/)).
2. Start Redis:
   ```bash
   redis-server
   ```
   Redis will be available at `localhost:6379`.

### Running the Worker (C#/.NET) Locally (No Docker)

1. Ensure .NET 7.0 SDK is installed.
2. Navigate to `worker`:
   ```bash
   cd worker
   dotnet restore
   dotnet run
   ```
   The worker will attempt to connect to Redis and Postgres when available.

### Running Postgres Locally (No Docker)

1. Install Postgres from [https://www.postgresql.org/download/](https://www.postgresql.org/download/).
2. Start Postgres, note the username and password (default `postgres`/`postgres`):
   ```bash
   # On many systems, Postgres runs as a service once installed.
   ```
   Postgres will be available at `localhost:5432`.

### Running the Result Service (Node.js) Locally (No Docker)

1. Ensure Node.js 18+ is installed.
2. Navigate to `result`:
   ```bash
   cd result
   npm install
   node server.js
   ```
   Access the results interface at [http://localhost:4000](http://localhost:4000).

**Note:** To get the entire system working end-to-end (i.e., votes flowing through Redis, processed by the worker, stored in Postgres, and displayed by the result app), you’ll need to ensure each component is running and that connection strings or environment variables point to the correct services.

---

## Running the Entire Stack in Docker

### Building and Running Individual Services

You can build each service with Docker and run them individually:

- **Vote (Python)**:
  ```bash
  docker build -t bacon21/vote:newest ./vote
  docker run --name vote -p 8080:80 bacon21/vote:newest
  ```
  Visit [http://localhost:8080](http://localhost:8080).

- **Redis** (official image, no build needed):
  ```bash
  docker run --name redis -p 6379:6379 redis:alpine
  ```

- **Worker (.NET)**:
  ```bash
  docker build -t bacon21/worker:latest ./worker
  docker run --name worker bacon21/worker:latest
  ```
  
- **Postgres**:
  ```bash
  docker run --name db -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15-alpine
  ```

- **Result (Node.js)**:
  ```bash
  docker build -t bacon21/result:newest ./result
  docker run --name result -p 8081:80 bacon21/result:newest
  ```
  Visit [http://localhost:8081](http://localhost:8081).

### Using Docker Compose

The easiest way to run the entire stack is via Docker Compose. From the project root directory:

```bash
docker compose up
```

This will:

- Build and run the vote, worker, and result services.
- Run Redis and Postgres from their official images.
- Set up networks, volumes, and environment variables so all services can communicate.

Visit [http://localhost:8080](http://localhost:8080) to vote and [http://localhost:8081](http://localhost:8081) to see results.

---

## Notes on Platforms (arm64 vs amd64)

If you’re on an arm64 machine (e.g., Apple Silicon M1/M2) and encounter issues with images or dependencies that assume amd64, you can use Docker `buildx`:

```bash
docker buildx build --platform linux/amd64 -t myorg/worker:latest ./worker
```

This ensures the image is built for the desired platform.

---

## AWS infrastructure and serving the apps

Provisioning aws infrastructure:
Ensure to have aws account and signed through aws cli.

```
cd terraform-files
run "terraform init"
run "terraform plan"
run "terraform apply"
```

This initializes terraform and prepares environment for running terraform files, displays planned deployments and provisions the necessary aws infrastructure to serve the applications in aws console.
For example; Creates bastion host for accessing the ec2 instances which are in private subnets to prevent access from the internet except through the ALB which through its' DNS the frontend is accessible in the browser.

Deploying all apps into them, ensuring they all communicate and accessible globally:

```
cd ansible-files
run "ansible-playbook -i inventory.ini install-docker.yml"
run "ansible-playbook -i inventory.ini frontend.yml"
run "ansible-playbook -i inventory.ini backend.yml"
run "ansible-playbook -i inventory.ini db.yml"
```

This installs docker in the respective ec2 instances(frontend, backend and database) for running the containers, deploys the apps in frontend, backend and database.

>>>>>>> 0683ff6 (init content)
