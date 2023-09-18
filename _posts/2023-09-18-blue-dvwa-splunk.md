---
layout: post
title: Blue DVWA Splunk
date: '2023-09-18 15:39:46 +0400'
categories: [Threat Hunting, SIEM]
tags: [Splunk, DVWA]
author: egycondor
commets: true
---

# Blue-DVWA-Splunk

Blue-DVWA-Splunk is a project that integrates DVWA (Damn Vulnerable Web Application), Splunk, and Apache logs for a comprehensive testing and monitoring environment. The project includes a Docker setup for running DVWA and Splunk, and configuration files for Splunk and Apache.

## Prerequisites

- Docker
- Docker Compose

## Installation

1. Clone this repository to your local machine.

    ```
    git clone https://github.com/egycondor/Blue-DVWA-Splunk.git
    cd Blue-DVWA-Splunk
    ```

2. Update the `docker-compose.yml` and `Dockerfile` files with your Splunk credentials and any other necessary changes.

3. Build the Docker images.

    ```
    docker-compose up --build
    ```

4. Access the DVWA application at `http://localhost:8081` and the Splunk dashboard at `http://localhost:8000`.

## Usage

1. Start the DVWA and Splunk containers.

    ```
    docker-compose up -d
    ```

2. Access the DVWA application at `http://localhost:8081` and the Splunk dashboard at `http://localhost:8000`.

3. Send requests to the DVWA application and observe the logs in the Splunk dashboard.

## Configuration

- `docker-compose.yml`: The Docker Compose file for running the DVWA and Splunk containers.
- `Dockerfile`: The Dockerfile for building the DVWA container with the necessary modifications for Splunk forwarding and Apache log configuration.
- `splunk-configs/props.conf`: The Splunk props.conf file for configuring field extraction and transformations.
- `splunk-configs/transforms.conf`: The Splunk transforms.conf file for configuring field transformations.

## Troubleshooting

- If you encounter an error during the Docker build process, check the Dockerfile and docker-compose.yml files for any configuration errors or missing dependencies.
- If the Apache logs are not being parsed correctly in Splunk, check the `props.conf` and `transforms.conf` files for any configuration errors.
- If the DVWA application is not working as expected, check the `config.inc.php` file for any configuration errors.

