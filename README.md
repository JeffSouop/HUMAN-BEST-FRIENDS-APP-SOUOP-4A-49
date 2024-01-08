# HumansBestFriend app? CATs or DOGs ?

A simple distributed application running across multiple Docker containers.

## Requirement

- The project need to be done inside a virtual machine that run docker and docker compose. Follow the docker documentation as we saw during the course for installation instructions if you don't have it yet.

## Getting started

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

## Tasks

1. Create a file called docker-compose.build.yml (not running containers). This file will be responsable for building the application images from the Dockerfile contents provided.

### For `worker` service

- The worker `depends_on` `redis` and `db`. Make use of below in your compose file

```shell
 depends_on:
   redis:
     condition: service_healthy
   db:
     condition: service_healthy
```

- The worker need to be inside `back-tier` network

```bash
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:7.0 as build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

WORKDIR /source
COPY *.csproj .
RUN dotnet restore -a $TARGETARCH

COPY . .
RUN dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

# app image
FROM mcr.microsoft.com/dotnet/runtime:7.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]
```

### For `vote` service

- map a volume at `/usr/local/app` that that is inside the container. This need to be a bind mount for example

```shell
volumes:
     - ./vote:/usr/local/app
```

- The `vote` service listen on port `80`. Feel free to expose for example `5002` outside.

- The `vote` service need to be inside two networks `front-tier` and `back-tier`
  ad below option for `vote`

```shell
healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
```

- Below is the Dockerfile of `vote` service

```shell
# Define a base stage that uses the official python runtime base image
FROM python:3.11-slim AS base

# Add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Define a stage specifically for development, where it'll watch for
# filesystem changes
FROM base AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
CMD ["python", "app.py"]

# Define the final stage that will bundle the application for production
FROM base AS final

# Copy our code from the current folder to the working directory inside the container
COPY . .

# Make port 80 available for links and/or publish
EXPOSE 80

# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
```

The `vote` app will be running at [http://localhost:5002](http://localhost:5002), and the `results` will be at [http://localhost:5001](http://localhost:5001).

### For `seed-data`

- Note: add for `seed`

```shell
profiles: ["seed"]
    depends_on:
      vote:
        condition: service_healthy
    restart: "no"
```

- It need to be inside `front-tier` network

- Below the Dockerfile

```shell
FROM python:3.9-slim

# add apache bench (ab) tool
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    apache2-utils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /seed

COPY . .

# create POST data files with ab friendly formats
RUN python make-data.py

CMD /seed/generate-votes.sh
```

### For `result`

- Inside the container the port is `80`. Feel free to expose outside with for example `5001`
  I strongly recommands you use below:

```shell
entrypoint: nodemon --inspect=0.0.0.0 server.js
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./result:/usr/local/app
    ports:
      - "5001:80"
      - "127.0.0.1:9229:9229"
```

- Below the Dockerfile for result

```shell
FROM node:18-slim

# add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl tini && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

# have nodemon available for local dev use (file watching)
RUN npm install -g nodemon

COPY package*.json ./

RUN npm ci && \
 npm cache clean --force && \
 mv /usr/local/app/node_modules /node_modules

COPY . .

ENV PORT 80
EXPOSE 80

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
```

2. The images that are build need to be published first in your publish docker registry and then in a private registry. Please make sure you have a frontend for the private registry as we saw in the course.

3. Create another file called `compose.yml` and it will be responsible of deploying the application and all the needed containers. Please make sure that the images are referencing the one in your private registry.

### For postgres `db`

- It need to be inside `back-tier` network
- Please make use of below in your compose file. Please use `postgres:15-alpine` for the postgres image

```shell
    volumes:
      - "db-data:/var/lib/postgresql/data"
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/postgres.sh
      interval: "5s"
```

- for the volume `db-data` make sure you create it in your compose file

### For `redis` service

Please make use of below in your compose file.

```shell
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/redis.sh
      interval: "5s"
```

- It need to be inside `back-tier` network

## Architecture

![Architecture diagram](architecture.png)

- A front-end web app in [Python](/vote) which lets you vote between two options
- A [Redis](https://hub.docker.com/_/redis/) which collects new votes
- A [.NET](/worker/) worker which consumes votes and stores them in…
- A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
- A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The `HumansBestFriend` application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.

# Submission

- You need to fork this project to your own public github repository
- You need to submit your own public github repository with the required files (`Dockerfile`, `compose.yml`, `docker-compose.build.yml` etc....).
- This need to be done inside a VMWare EXSI host with 3 VMs. One will act as the master and the two others workers worker1 worker2.
- Deploy the app first using docker compose
- Deploy the app inside a kubernetes cluster
- Make sure the kubernetes components are up and running (api-server, kubelet, kubeproxy and so on...)
- This need to be in a team of 3 MAXIMUM. No single submission is allowed. Please do not use docker desktop to do the job
- Take all the necessary screenshots that will showcase your work
- The moodle is `G-[YOUR-CREW]`. You need to submit before the deadline.
- Make sure your team members name is visible inside your pdf file

### Usefull links

- https://kubernetes.io/docs/setup/production-environment/tools/
- https://docs.docker.com/engine/install/ubuntu/
- https://docs.docker.com/compose/gettingstarted/
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
