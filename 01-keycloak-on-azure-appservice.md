# Deploy KeyCloak on public HTTPS URL on Azure AppService


## What

[KeyCloak](https://www.keycloak.org/) is *open-source Identity and Access Manager* that makes it easy to **create your own SAML or OpenId Connect based authentication service** for your apps and services. 
It supports strong authentication, user sources federation (e.g. LDAP), use of various other identity providers (like social logins or AzureAD). It allows managing users and fine-authorization.

It can be authenticated with many Check Point products including
* [Infinity Portal](https://portal.checkpoint.com/signin)
* [Harmomy Connect](https://www.checkpoint.com/harmony/connect-sase/)
* [Security Gateways](https://www.checkpoint.com/quantum/next-generation-firewall/) including Remote Access VPN client, client-less Mobile Access Blade or Identity Awareness


## Prerequisites

* Azure subscription


## Design goals

* fully-automated deployment to cloud
* HTTPS service on public Internet
* cost-effective for lab/tests


## Quick Start

* login to [Azure Shell](https://shell.azure.com)
* review deployment script `curl -s https://bit.ly/deploy-keycloack-azure-webapp-sh`
* deploy on using `curl -s https://bit.ly/deploy-keycloack-azure-webapp-sh | bash`


## Click-ops approach

This automation is inspired by video by CoreWreck's video [Running a dev keycloak instance in Azure with HTTPS in few steps](https://www.youtube.com/watch?v=neHFkd8c-gc) 


## In more detail - step by step

### Deployment environment

Please start in authenticated [Azure Shell](https://shell.azure.com) with valid Azure subscription.

### Create unique project name and hostname

```bash
PROJECT_TAG=$(hexdump -vn16 -e'4/4 "%08X" 1 "\n"' /dev/urandom | cut -c-6); echo $PROJECT_TAG

PROJECT_NAME="keycloak-$PROJECT_TAG"; 
echo $PROJECT_NAME
```

### Create resource group in relevant region and App Service Plan

```bash
az group create --name "$PROJECT_NAME" --location "West Europe"

az appservice plan create --name appservice-plan-$PROJECT_NAME --resource-group "$PROJECT_NAME" --sku B2 --is-linux
```
