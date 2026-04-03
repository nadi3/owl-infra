# Owl-infra: Personal self-hosted tools

Welcome to the infrastructure repository for my personal self-hosted cloud environment. This repository contains the Infrastructure as Code (IaC) required to deploy and manage a secure, containerized set of services using Docker Compose.

## Software Stack

This infrastructure relies on a modern, open-source stack designed for security, privacy, and performance:

* **Network and ingress:**
  *  [Cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (Secure outbound tunnel, zero open ports)
  * [Traefik v3](https://traefik.io/traefik/) (Dynamic Reverse Proxy)

* **Identity and access management:** 
  * [Authentik](https://goauthentik.io/) (Centralized SSO, SAML, and OIDC provider)

* **Services:**
  * [Nextcloud AIO](https://github.com/nextcloud/all-in-one) (Personal cloud, file sync, and collaboration)
  * [SonarQube](https://www.sonarsource.com/products/sonarqube/) (Continuous Code Quality and Security Analysis)

## Hardware

This stack is currently optimized to run on lightweight, energy-efficient hardware:
* **Host:** Raspberry Pi 4 5
* **Storage:** NVMe SSD (good for database performance and Elasticsearch stability required by SonarQube)
* **OS:** Linux (Raspberry Pi OS lite) with modified kernel parameters (`vm.max_map_count=524288`) for Elasticsearch.

## Repository structure and secrets management

To maintain absolute security and prevent secret leakage, this repository follows a strict `env` variable philosophy. **No passwords or API keys are hardcoded in this repository.**

```text
owl-infra/
├── docker-compose.yml		 # Cloudflare tunnel & Traefik
├── authentik/             # SSO
│   ├── docker-compose.yml
│   └── .env.example
├── nextcloud/             # File storage
│   ├── docker-compose.yml
│   └── .env.example
└── sonarqube/             # Code analysis
    ├── docker-compose.yml
    └── .env.example
```

> **Note:** Every directory contains a `.env.example` file. You must duplicate this file, rename it to `.env`, and fill in your secure passwords before deploying.

## Deployment guide

If you are restoring this infrastructure from scratch, follow these steps:

1. **Clone the repository:**

   ```bash
   git clone git@github.com:nadi3/owl-infra.git
   cd owl-infra
   ```

2. **Prepare the network:**

   Create the external Docker network that allows the proxy to talk to the containers:

   ```bash
   docker network create proxy-net
   ```

3. **Configure Secrets:**

   Go into each folder, copy the example variables, and fill them with your secure data:

   ```
   cp authentik/.env.example authentik/.env
   # Edit the .env file with nano/vim
   ```

4. **Spin up the services:**

   Launch the services folder by folder, starting with the core infrastructure:

   ```bash
   cd docker compose up -d
   cd ../authentik && docker compose up -d
   cd ../nextcloud && docker compose up -d
   cd ../sonarqube && docker compose up -d
   ```



## Post-deployment configuration

This repository strictly handles the **container deployment**. It does not automatically configure the internal logic of the applications. Once the containers are running, you must manually link the services together using their respective administrative interfaces:

* **Authentik:** You will need to create Providers and Applications to enable SSO.
  * *See Docs:* [Authentik Application Integration](https://docs.goauthentik.io/docs/providers/)
* **Nextcloud:** Must be connected to Authentik via OpenID Connect (OIDC).
  * *See Docs:* [Nextcloud OIDC integration](https://docs.goauthentik.io/integrations/services/nextcloud/)
* **SonarQube:** Must be connected to Authentik via SAML, and the base URL must be updated in the settings.
  * *See Docs:* [SonarQube SAML Authentication](https://docs.sonarsource.com/sonarqube-server/latest/instance-administration/authentication/saml/overview/)



## Security posture (Zero Trust)

This infrastructure is built with a Zero-Trust mindset. **There are absolutely no incoming ports open on the local home router.** All external traffic is securely routed through a Cloudflare Tunnel directly to Traefik. Furthermore, critical applications are placed behind Authentik's Forward Auth middleware, meaning unauthenticated users cannot even reach the login pages of the underlying services.



## Disclaimer

This repository is public for educational purposes, to share architectural ideas, and to serve as my personal backup. Because it is tailored to my specific home network and domain names, **I do not accept Pull Requests (PRs)**. However, feel free to fork this project and adapt it to build your own infrastructure.