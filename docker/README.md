# Docker Task

## Application
A simple application that displays: "Construction Technect DevOps Test"

## Dockerfile
See the Dockerfile in the repository root.

## Docker Commands Used
docker build -t construction-technect-app .
docker run -d -p 3000:3000 --name technect-container construction-technect-app
docker ps
docker logs technect-container
docker stop technect-container
docker start technect-container

## Explanations

**What is a Docker image?**
A Docker image is a read-only template containing the application code, dependencies, and configuration needed to run the app.

**What is a Docker container?**
A container is a running instance of a Docker image — a lightweight, isolated environment running the application.

**Why would Docker be useful for a SaaS application?**
It ensures consistency across environments, speeds up deployment, isolates dependencies, and makes scaling and running the app on any machine or cloud provider easier.

**What happens if the application container stops?**
The app becomes inaccessible. Data not saved in a persistent volume is lost. The container can be restarted using `docker start <container-name>`.