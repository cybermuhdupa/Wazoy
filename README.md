# VEnvoy: Envoy + Wazuh for Visibility Enhancement (VE)

## Overview
VEnvoy is a Dockerized Envoy Proxy with a pre-configured Wazuh Agent, designed to enhance the visibility and observability of a CTF challenge. VEnvoy automatically captures HTTP traffic, including both body and headers, and directly transports this data to Wazuh for examination if any issues arise with the challenge.

## Architecture 

![alt text](./graphics/architecture.png)

### Basic Principles

VEnvoy operates as a Man-in-the-Middle (MITM) proxy, but not just for HTTP request. VEnvoy currently supports TCP, HTTP, and UDP connections.

For example, your CTF challenge runs on port `:5000`, and you configure VEnvoy to listen on port `:8000`. Envoy will intercept and log all traffic, then redirect them to the `:5000` service. You can choose any port for VEnvoy to listen on, not just `:8000`.

All traffic passing through Envoy is recorded in two files: *access.log* and *alltaps.json*. 
- access.log: Contains a record of all requests made to the challenge's service.
- alltaps.json: Provides a detailed log of traffic data, including body and header information (available only for HTTP traffic).

Those files will automatically be transported to Wazuh and accessible via Wazuh Dashboard.

## Getting Started
### 1. Setup Your Challenge Container
Well, just do your thing.\
Spawn spawn bang bang challenge development.\
¯\\_(ツ)_/¯

### 2. Sidecar VEnvoy to Challenge Container

You can obtain a prebuilt version of VEnvoy from [siahaan/venvoy](https://hub.docker.com/repository/docker/siahaan/venvoy) on Docker Hub.

To use VEnvoy for your CTF challenge, adjust your `docker-compose.yml` to include the VEnvoy container(s). As a sidecar deployment, **one CTF challenge container will need one VEnvoy container**.

```
version: '3'

services:
  
  # CTF Challenge Container
  app:
    build: ./app
    expose:
      - 5000
  
  # VEnvoy Container
  venvoy:
    image: siahaan/venvoy
    environment:
      WAZUH_MANAGER_IP: <ip_to_wazuh_server>
      APP_HOST: app
      APP_PORT: 5000
      ENTRY_PORT: 8082 
    ports:
      - <anyport>:8082
    
```

To start, there are few basic things to configure:
- WAZUH_MANAGER_IP: IP address of your Wazuh server.
- APP_HOST: Hostname or service name of your challenge container (e.g., app).
- APP_PORT: Port on which the challenge service is running internally (e.g., 5000).
- ENTRY_PORT:  Port on which VEnvoy listens (e.g., 8082).

Ensure that the `ENTRY_PORT` specified in the environment variables (e.g., `8082`) matches the port mapping in the `docker-compose.yml` file (e.g., `<anyport>:8082`).

### 3. Install Custom Decoder to Wazuh Dashboard

To process VEnvoy logs in Wazuh, upload custom decoders and rules into Wazuh Dashboard.

**Access the Wazuh Dashboard**

Open your browser, navigate to the Wazuh Dashboard, and log in.

**Upload Custom Decoders**

- Go to `Settings > Decoders`.
- Click `Upload` or `Add Decoder`.
- Upload all files from the `./wazuh/decoders` directory to Wazuh Dashboard.

**Upload Custom Rules**

- Go to `Settings > Rules`.
- Click `Upload` or `Add Rule`.
- Upload all files from the `./wazuh/rules` directory to Wazuh Dashboard.

**Apply Changes**

Restart the Wazuh manager for changes to take effect.

## Understanding Alerts


