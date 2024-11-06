# How I setup Sql+SSIS in Docker

## The system

- Proxmox server
  - Ubuntu Server VM
    - Docker Compose

## The Tools

- Neovim [Text Editor/IDE]
- SSH
- Azure Data Studio (But do I wish I had DataGrip)

## Install sqlcmd (Prerequisite)

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc

sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/20.04/prod.list)"

sudo apt-get update
sudo apt-get install sqlcmd
```

## Setup Docker Compose

### Original Commands from Docs

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrong@Passw0rd>" \
   -p 1433:1433 --name sql1 --hostname sql1 \
   -d \
   mcr.microsoft.com/mssql/server:2022-latest
```

### Composerize.com Output

name: <your project name>
services:
server:
environment: - ACCEPT_EULA=Y - MSSQL_SA_PASSWORD=<YourStrong@Passw0rd>
ports: - 1433:1433
container_name: sql1
hostname: sql1
image: mcr.microsoft.com/mssql/server:2022-latest

```bash
docker compose up -d (alias = dcu)
```

### Load Adventureworks Sample Data

```bash
sqlcmd create mssql --accept-eula --using https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2022.bak

# Test
sqlcmd query "SELECT @@version"
```
