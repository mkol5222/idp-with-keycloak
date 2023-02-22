# Deploy KeyCloak on public HTTPS URL on Azure App Service


## What

[KeyCloak](https://www.keycloak.org/) is *open-source Identity and Access Manager* that makes it easy to **create your own SAML or OpenId Connect based authentication service** for your apps and services. 
It supports strong authentication, user sources federation (e.g. LDAP), use of various other identity providers (like social logins or AzureAD). It allows managing users and fine-authorization.

It can be used to authenticate and authorize many Check Point products including
* [Infinity Portal](https://portal.checkpoint.com/signin)
* [Harmomy Connect](https://www.checkpoint.com/harmony/connect-sase/)
* [Security Gateways](https://www.checkpoint.com/quantum/next-generation-firewall/) including Remote Access VPN client, client-less Mobile Access Blade or Identity Awareness


## Prerequisites

* Azure subscription


## Design goals

* fully-automated deployment to cloud
* HTTPS service on public Internet
* cost-effective for lab/tests


## Option 1: Quick Start

Use followinng or continue in step by step section manually:
* login to [Azure Shell](https://shell.azure.com)
* review deployment script `curl -s -L https://bit.ly/deploy-keycloack-azure-webapp-sh`
* deploy on using `curl -s -L https://bit.ly/deploy-keycloack-azure-webapp-sh | bash`


## Option 2: Click-ops approach

This automation is inspired by video by CoreWreck's video [Running a dev keycloak instance in Azure with HTTPS in few steps](https://www.youtube.com/watch?v=neHFkd8c-gc) 


## Option 3: In more detail - step by step

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


### Application container using Docker Compose

We will deploy as multi-container orchestrated with Docker Compose. Notice data will persist between restarts thanks to volume:
```bash
tee /tmp/$PROJECT_NAME-docker-compose.yml << 'EOF'
version: 3
services:
  keycloak:
    image: jboss/keycloak:latest
    volumes:
      - ${WEBAPP_STORAGE_HOME}/data:/opt/jboss/keycloak/standalone/data
    restart: always
EOF
```

### Create App Service

Based on temporary docker-compose.yml file we create new AppService

```bash
az webapp create --resource-group "$PROJECT_NAME" --plan appservice-plan-$PROJECT_NAME --name $PROJECT_NAME --multicontainer-config-type compose --multicontainer-config-file /tmp/$PROJECT_NAME-docker-compose.yml
```


### Configure KeyClock AppService

```bash
# obtain app's hostname
KC_HOST=$(az webapp show --name $PROJECT_NAME --resource-group $PROJECT_NAME | jq -r .defaultHostName); echo $_KC_HOST

# generate random password
KC_PASS=$(hexdump -vn16 -e'4/4 "%08X" 1 "\n"' /dev/urandom); echo $KC_PASS

# add configuration settings for our app
az webapp config appsettings set --name $PROJECT_NAME --resource-group $PROJECT_NAME --settings KEYCLOAK_FRONTEND_URL="https://$KC_HOST/auth" KEYCLOAK_USER=kcadmin KEYCLOAK_PASSWORD="$KC_PASS" WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
```


### Restart app to apply the settings

```bash
az webapp restart --name $PROJECT_NAME --resource-group $PROJECT_NAME
```

### Get login details

Will output KeyCloak login URL and user account:
```bash
echo "visit https://$KC_HOST/auth/admin and login as kcadmin / $KC_PASS"
```

Continue in KeyCloak with new realm creation and integrations configuration.
