# Oracle 23ai Free on Arch Linux (via Docker)

This guide explains how to install, run, and manage **Oracle Database 23ai Free** inside a Docker container on **Arch Linux**.

---

## Prerequisites

Make sure you have Docker installed and running:

```bash
sudo pacman -S docker
sudo systemctl enable --now docker
```

Check that it works:

```bash
docker --version
```

---

## Step 1: Pull Oracle 23ai Image

Log in to Oracle's Container Registry (first-time only):
[https://container-registry.oracle.com](https://container-registry.oracle.com)

Then pull the image:

```bash
docker pull container-registry.oracle.com/database/free:latest
```

---

## Step 2: Run the Oracle 23ai Container

Create a persistent data directory:

```bash
sudo mkdir -p ~/oracle_data
sudo chmod 777 ~/oracle_data
```

Run the container:

```bash
docker run -d \
  --name oracle23ai \
  -p 1522:1521 -p 8085:5500 \
  -e ORACLE_PWD=StrongPass123 \
  -v ~/oracle_data:/opt/oracle/oradata \
  container-registry.oracle.com/database/free:latest
```

* `1522` → host port for SQL connections (maps to 1521 in container)
* `8085` → host port for web UI (maps to 5500 in container)
* `ORACLE_PWD` → default password for SYS, SYSTEM, and PDBADMIN
* `~/oracle_data` → persistent data folder on the host

Follow logs to monitor startup:

```bash
docker logs -f oracle23ai
```

Wait for:

```
#########################
DATABASE IS READY TO USE!
#########################
```

---

## Step 3: Connect to the Database

### Inside the container

```bash
docker exec -it oracle23ai bash
sqlplus system/StrongPass123
```

### From host (after installing client)

```bash
yay -S oracle-instantclient-basic oracle-instantclient-sqlplus
sqlplus system/StrongPass123@//localhost:1522/FREEPDB1
```

---

## Step 4: Create a User with Full Privileges

Once connected to SQL*Plus:

```sql
ALTER SESSION SET CONTAINER=FREEPDB1;
CREATE USER rik IDENTIFIED BY StrongPass123;
GRANT ALL PRIVILEGES TO rik;
ALTER USER rik QUOTA UNLIMITED ON USERS;
```

You can now log in as:

```bash
sqlplus rik/StrongPass123@//localhost:1522/FREEPDB1
```

---

## Step 5: Create Shell Aliases

Add these to your `~/.bashrc` or `~/.zshrc`:

```bash
# Oracle 23ai Docker aliases
alias oracle-start='sudo docker start -ai oracle23ai'
alias oracle-stop='sudo docker stop oracle23ai'
alias oracle-status="sudo docker ps --filter name=oracle23ai"
alias oracle-logs='sudo docker logs -f oracle23ai'
```

Reload your shell:

```bash
source ~/.bashrc
```

Now you can manage your Oracle DB easily:

```bash
oracle-start     # Start container
oracle-stop      # Stop container
oracle-status    # Show running status
oracle-logs      # Follow logs
```

---

## Web Interface

Once running, open:

```
https://localhost:8085/em
```

Log in as:

```
Username: system
Password: StrongPass123
```

---

## Optional: Auto-Start on Boot

To have the container start automatically with Docker:

```bash
sudo docker update --restart unless-stopped oracle23ai
```

---

### You now have a fully functional Oracle 23ai Free database running on Arch Linux — isolated, persistent, and ready for development.
