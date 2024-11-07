# How I setup Sql+SSIS in Docker

## The system

- Proxmox server
  - Ubuntu Server VM
    - Docker Compose

## The Tools

- Neovim [Text Editor/IDE]
- SSH
- ~Azure Data Studio~ (But do I wish I had DataGrip)
- Visual Studio (SSDT)

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

```yml
name: <your project name>
services:
server:
environment: - ACCEPT_EULA=Y - MSSQL_SA_PASSWORD=<YourStrong@Passw0rd>
ports: - 1433:1433
container_name: sql1
hostname: sql1
image: mcr.microsoft.com/mssql/server:2022-latest
```

```bash
docker compose up -d (alias = dcu)
```

## Lesson 1

### Load Adventureworks Sample Data

```bash
sqlcmd create mssql --accept-eula --using https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorksDW2022.bak

# Test
sqlcmd query "SELECT @@version"
```

Installed Azure Data Studio and connected it to my SQL Server
Installed SQL Database Projects extension
Ran Database: New command to create the SSIS Learning project

### Lesson 1-1

Just creating the project and I'm already straying a bit.
May need to come back and not use SDK style and ADS.
But for now I'm going to trying to translate.

### Lesson 1-1 (Again)

This time with Visual Studio.
I didn't want to have to install it on this machine but here we go!
Also had to install SSIS Projects for VS.

### Lesson 1-2

Added a Flat File Connection Manager and setup the data types for it.

### Lesson 1-3

Setup DBConnection

### Lesson 1-4

Add a DataFlow

### Lesson 1-5

Add Flat File source to the Data Flow

### Lesson 1-6

Ran into a small issue here.
The way I setup the SQL Server it made a second context.
This context runs on port 1434. That port was taken on the real machine.
So I've added 1444:1434 to the docker config

NOPE!
The above sqlcmd create command spins up a new docker container on it's own.
This means the 1434 port was already being exposed.
I pulled down the docker compose version and will just run with what it made.

### Lesson 1-7

Setup the destination for the two flows from 1-6

### Lesson 1-8

Formatting

### Lesson 1-9

Ran the package and it worked.
Verified with the following

```sql
SELECT * FROM NewFactCurrencyRate2
```
