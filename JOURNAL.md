# How I setup Sql+SSIS in Docker

<!--toc:start-->
- [How I setup Sql+SSIS in Docker](#how-i-setup-sqlssis-in-docker)
  - [The system](#the-system)
  - [The Tools](#the-tools)
  - [Install sqlcmd (Prerequisite)](#install-sqlcmd-prerequisite)
  - [Setup Docker Compose](#setup-docker-compose)
    - [Original Commands from Docs](#original-commands-from-docs)
    - [Composerize.com Output](#composerizecom-output)
    - [Load Adventureworks Sample Data](#load-adventureworks-sample-data)
  - [Lesson 1](#lesson-1)
    - [Lesson 1-1](#lesson-1-1)
    - [Lesson 1-1 (Again)](#lesson-1-1-again)
    - [Lesson 1-2](#lesson-1-2)
    - [Lesson 1-3](#lesson-1-3)
    - [Lesson 1-4](#lesson-1-4)
    - [Lesson 1-5](#lesson-1-5)
    - [Lesson 1-6](#lesson-1-6)
    - [Lesson 1-7](#lesson-1-7)
    - [Lesson 1-8](#lesson-1-8)
    - [Lesson 1-9](#lesson-1-9)
<!--toc:end-->

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

### Load Adventureworks Sample Data

```bash
sqlcmd create mssql --accept-eula --using https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorksDW2022.bak

# Test
sqlcmd query "SELECT @@version"
```

## Lesson 1

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

## Lesson 2

Add Looping

### Lesson 2-1

Copy/Paste Lesson 1.dtsx and generate a new ID

### Lesson 2-2

Create a ForEach Container.
Change to the ForEachFile Enum and browse to the Sample data
We also added a variable named varFileName with no value

### Lesson 2-3

Convert the Flat File connection to a package connection
This allow us to use the variable above as our connection string

### Lesson 2-4

Test the package.
Worked fine first run.

## Lesson 3

Add Logging. 
Thank goodness the last package just ran with no output at all

### Lesson 3-1

Copy the package again
This time I checked the ID after copying and it was already changed
No need to Generate a new ID as the tutorial says

However the name of the Package (not the file) is not automatically changed to match the new file name

### Lesson 3-2

Extensions > SSIS > Logging
Created a File logger with two events selected

### Lesson 3-3

Ran the package and the log file was created
However the Log Events window seem broken
It's either see through or completely white
I can't find anyone else with this issue and simple troubleshooting short of rebooting has not corrected it

## Lesson 4

I'm not even going to mention 4-1
We all know the first step

### Lesson 4-2

Created Currency_BAD.txt by copying Currency_VEB.txt
and then running the `:%s/VEB/BAD` command in Neovim

Ran package and it failed successfully

### Lesson 4-3

Added a script node and set up its column data
In another VS instance I added a single line of C#

Oops! I was in Lesson 2 still due to trying to debug the logging issue.
Reviewed any changes to Lesson 2 in git but found that I hadn't been committing
after each lesson as I should be. :-(

### Lesson 4-4

Added a Flat File Destination for error output
Tried to run package but got error
Seems that when I migrated the Script Node it's script didn't come with it
Actually seems I just needed to build it again!

### Lesson 4-5

Project now successfully fails with Error Output

Git commit!

## Lesson 5

### Lesson 5-2

Added variable

Breaking Change:
I had apparently added my sql connection on a project level
This prevented me from converting the project to package deployment model

Lessons 1-4 are, I believe now all broken.

Setup the XML config file for our folder variable

### Lesson 5-3

Used Neovim to edit the XML file to set the value.
Note to Self: Need an XML LSP for Neovim

### Lesson 5-4

Before running the test I need to fix up the breaking change from 5-2.
I expected this to just be setting the source again but the custom SQL script
was also lost and I had to go retrieve that!

Now Lesson 5 runs fine.
RIP Lessons 1-4
