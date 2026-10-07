# Odoo 19 — Windows 11 Local Setup

A simple guide for setting up **Odoo 19 Community** locally on **Windows 11**, using **Python virtual environment** and **PostgreSQL 17 running with Docker**.

---

## 1. Requirements

Make sure these are installed:

- Windows 11
- Git
- Python 3.12
- Docker Desktop
- PostgreSQL 17 — running through Docker

Check the installations:

```powershell
git --version
python --version
docker --version
```

---

# 2. Clone Odoo 19

Clone the Odoo 19 Community source code:

```powershell
git clone --depth 1 --branch 19.0 https://github.com/odoo/odoo.git
```

Enter the Odoo directory:

```powershell
cd odoo
```

---

# 3. Create Python Virtual Environment

Create a virtual environment:

```powershell
python -m venv venv
```

If you have multiple Python versions, you can specifically use Python 3.12:

```powershell
py -3.12 -m venv venv
```

Activate the virtual environment:

```powershell
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks the activation script, run:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Then activate again:

```powershell
.\venv\Scripts\Activate.ps1
```

You should now see something similar to:

```text
(venv) PS C:\...\odoo>
```

---

# 4. Install Python Dependencies

Upgrade the Python packaging tools:

```powershell
python -m pip install --upgrade pip setuptools wheel
```

Install Odoo's required Python packages:

```powershell
pip install -r requirements.txt
```

> **Note:** Keep the virtual environment activated while installing and running Odoo.

---

# 5. PostgreSQL 17 with Docker

Instead of installing PostgreSQL directly on Windows, we use Docker.

### Pull PostgreSQL 17

```powershell
docker pull postgres:17
```

### Create a persistent Docker volume

This keeps PostgreSQL data even if the container is removed:

```powershell
docker volume create odoo-postgres-data
```

### Create the PostgreSQL container

Run:

```powershell
docker run -d `
  --name odoo-postgres `
  -e POSTGRES_USER=odoo `
  -e POSTGRES_PASSWORD=odoo `
  -e POSTGRES_DB=postgres `
  -p 5432:5432 `
  -v odoo-postgres-data:/var/lib/postgresql/data `
  postgres:17
```

### Check the container

```powershell
docker ps
```

Expected result:

```text
odoo-postgres
```

---

# 6. Test PostgreSQL

Connect to PostgreSQL inside the Docker container:

```powershell
docker exec -it odoo-postgres psql -U odoo -d postgres
```

If successful, you should see:

```text
postgres=#
```

Exit PostgreSQL:

```sql
\q
```

---

# 7. Create Odoo Configuration

Create an Odoo configuration file:

```powershell
notepad odoo.conf
```

Add:

```ini
[options]

admin_passwd = admin

db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo

addons_path = C:\Users\hp\Desktop\odoo project\odoo\addons,C:\Users\hp\Desktop\odoo project\custom-addons

http_port = 8069
```

### Configuration explanation

| Setting | Description |
|---|---|
| `admin_passwd` | Odoo database manager password |
| `db_host` | PostgreSQL server |
| `db_port` | PostgreSQL port |
| `db_user` | PostgreSQL username |
| `db_password` | PostgreSQL password |
| `addons_path` | Odoo core and custom modules |
| `http_port` | Odoo web server port |

> **Important:** Change the `addons_path` if your project is stored in a different folder.

---

# 8. Recommended Project Structure

A simple structure:

```text
odoo project/
│
├── odoo comunity/
│   ├── odoo/
│   │   ├── addons/
│   │   ├── odoo-bin
│   │   ├── requirements.txt
│   │   └── venv/
│   │
│   └── odoo.conf
│
└── custom-addons/
    ├── module_1/
    ├── module_2/
    └── ...
```

Your `addons_path` should contain both:

```text
Odoo core addons
+
Your custom addons
```

---

# 9. Start Odoo

Go to the Odoo directory:

```powershell
cd "C:\Users\hp\Desktop\odoo project\odoo comunity\odoo"
```

Activate the virtual environment:

```powershell
.\venv\Scripts\Activate.ps1
```

Start Odoo:

```powershell
python odoo-bin -c "..\odoo.conf"
```

If everything is working, Odoo should start on:

```text
http://localhost:8069
```

Open the URL in your browser.

---

# 10. Stop Odoo

In the terminal running Odoo, press:

```text
Ctrl + C
```

This stops the Odoo server.

The PostgreSQL Docker container continues running.

---

# 11. Start PostgreSQL Again

If you restart your computer, check the container:

```powershell
docker ps
```

If `odoo-postgres` is not running:

```powershell
docker start odoo-postgres
```

Check again:

```powershell
docker ps
```

---

# 12. Useful Docker Commands

### Check running containers

```powershell
docker ps
```

### Check all containers

```powershell
docker ps -a
```

### Start PostgreSQL

```powershell
docker start odoo-postgres
```

### Stop PostgreSQL

```powershell
docker stop odoo-postgres
```

### Restart PostgreSQL

```powershell
docker restart odoo-postgres
```

### View PostgreSQL logs

```powershell
docker logs odoo-postgres
```

### Open PostgreSQL shell

```powershell
docker exec -it odoo-postgres psql -U odoo -d postgres
```

---

# 13. Useful Odoo Commands

### Start Odoo

```powershell
python odoo-bin -c "..\odoo.conf"
```

### Update a module

```powershell
python odoo-bin -c "..\odoo.conf" -u module_name
```

### Update all modules

```powershell
python odoo-bin -c "..\odoo.conf" -u all
```

### Stop Odoo

```text
Ctrl + C
```

---

# 14. Quick Start

After the initial setup, the normal workflow is:

### Terminal 1 — PostgreSQL

```powershell
docker start odoo-postgres
```

### Terminal 2 — Odoo

```powershell
cd "C:\Users\hp\Desktop\odoo project\odoo comunity\odoo"
```

Activate the environment:

```powershell
.\venv\Scripts\Activate.ps1
```

Start Odoo:

```powershell
python odoo-bin -c "..\odoo.conf"
```

Then open:

```text
http://localhost:8069
```

---

# 15. Troubleshooting

## PostgreSQL connection error

Check that PostgreSQL is running:

```powershell
docker ps
```

Check the PostgreSQL logs:

```powershell
docker logs odoo-postgres
```

Test the database:

```powershell
docker exec -it odoo-postgres psql -U odoo -d postgres
```

Verify the Odoo configuration:

```ini
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo
```

---

## Python virtual environment does not activate

Run:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Then:

```powershell
.\venv\Scripts\Activate.ps1
```

---

## Port 5432 is already being used

Check which process is using port 5432:

```powershell
netstat -ano | findstr :5432
```

If another PostgreSQL installation is already using port `5432`, stop it or change the Docker port.

---

## Port 8069 is already being used

Check:

```powershell
netstat -ano | findstr :8069
```

You can also change the Odoo port in `odoo.conf`:

```ini
http_port = 8070
```

Then open:

```text
http://localhost:8070
```

---

# 16. Development Workflow

For normal development:

```text
GitHub
   │
   ▼
Odoo 19 Source Code
   │
   ├── Odoo Core
   │
   ├── Custom Addons
   │
   └── Python Virtual Environment
             │
             ▼
       Odoo Server :8069
             │
             ▼
      PostgreSQL 17
         Docker :5432
```

When developing a custom module:

```text
1. Create/edit module
        ↓
2. Restart Odoo
        ↓
3. Upgrade module
        ↓
4. Test in browser
        ↓
5. Commit changes
        ↓
6. Push to GitHub
```

---

# 17. Git Commands

Check the current changes:

```powershell
git status
```

Add changes:

```powershell
git add .
```

Create a commit:

```powershell
git commit -m "Update Odoo configuration"
```

Push to GitHub:

```powershell
git push
```

Pull the latest changes:

```powershell
git pull
```

---

## Setup Complete

You now have:

- **Odoo 19 Community**
- **Python virtual environment**
- **PostgreSQL 17**
- **PostgreSQL running in Docker**
- **Persistent PostgreSQL volume**
- **Custom addons directory**
- **Git/GitHub workflow**
- **Odoo running on port `8069`**

Main URLs:

```text
Odoo:
http://localhost:8069

PostgreSQL:
localhost:5432
```
