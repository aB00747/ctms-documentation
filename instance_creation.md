# CTMS/SOMS Development Setup & Operations Guide

## Table of Contents

- [Repository Setup](#repository-setup)
- [Environment Configuration](#environment-configuration)
- [Database Operations](#database-operations)
- [New Instance Setup](#new-instance-setup)
- [Server Management](#server-management)
- [File Transfer Commands](#file-transfer-commands)
- [Troubleshooting](#troubleshooting)F
- [Git Operations](#git-operations)
- [MCP Server Setup](#mcp-server-setup-for-windsurf)

---

## Repository Setup

### CTMS Repository

```bash
git clone -b main git@github.com:RealTime-Software-Solutions/ctms.git --sitename--
# Example: esrc251-ctms
```

### SOMS Repository

```bash
git clone -b main git@github.com:RealTime-Software-Solutions/soms.git esrc2230
```

### Migration Repository

```bash
git clone -b main git@github.com:RealTime-Software-Solutions/migrations.git eng-migration
```

---

## Environment Configuration

### Quick Setup Commands

**For CTMS:**

```bash
bash /srv/http/ctms/abhishek/setup_environment.sh
```

**For SOMS:**

```bash
bash /srv/http/ctms/abhishek/setup_environment_soms.sh
```

### Manual Setup Process

```bash
cd --sitename--
git config core.fileMode false
cd app/js

# Remove existing directories
rm -rf formbuilder
rm -rf formgenerator
rm -rf web-integration
cd ../..

# Run provisioning
php util/provision.php

# Set permissions
sudo chown -Rf abhishek:devops .
sudo chmod -Rf 775 .
git config core.fileMode false

# Configure submodules
cd app/js/formbuilder
git config core.fileMode false
git checkout main

cd ../formgenerator
git config core.fileMode false
git checkout main

cd ../web-integration
git config core.fileMode false
cd ../../..
```

### Document Upload Permissions

```bash
rm -rf ./documents
ln -s /home/realtime/docstore/documents/dyn ./documents
```

### PDF Viewer Setup

```bash
rm -rf pdfviewer
ln -s app/js/formbuilder/public/viewer ./pdfviewer
```

---

## Database Operations

### Database Credentials

- **Host:** `us-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com`
- **Username:** `sbox`
- **Password:** `sbox`

### Connection Command

```bash
mysql -hus-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com -u sbox -p
```

### Database Import Process

1. **Prepare SQL file (remove DEFINER clauses):**

```bash
sed -i 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/' tmp_brpsmiami_20250902_093650_anonymized.sql
```

1. **Import examples:**

```bash
# CTMS Database
mysql -usbox -psbox -hus-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com ctms4098_new < tmp_brpsmiami_20250902_093650_anonymized.sql

# Optimus Database
mysql -usbox -psbox -hus-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com esrc2836_optimus < tmp_optimus_20250613_110145_anonymized.sql

# UCC Database
mysql -usbox -psbox -hus-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com esrc2228_ucc < tmp_ucctmso_20250512_083421_anonymized.sql
```

### Database Backup & Restore

**Create backup:**

```bash
mysqldump -u'username' -p'password' -h 127.0.0.1 --single-transaction 'dbname' > 'dbname'.sql
```

**Prepare for import:**

```bash
sed -i 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/' 'dbname'.sql
```

**Import to RDS:**

```bash
mysql -u'username' -p'password' -h'dbhost' 'dbname' < 'db_dump.sql'
```

---

## New Instance Setup

### Engage Instance Configuration

**Reset admin user:**

```sql
UPDATE tblUser 
SET Locked = 0, 
    FailedAttempts = 0, 
    Password = '$2y$10$5s2bu5r/tq/hFwyo3uKlte59QU36uQkuTR2kZ6iCapjacdDlt7wMi', 
    ResetPassword = 0  
WHERE UserName = 'admin';
```

**Update configuration:**

```sql
UPDATE tblConfiguration 
SET HostCTMS = 'https://us9-74.realtime-host01.com/al-engage';

UPDATE tblConfigurationOption 
SET OptionValue = 'https://dev.mystudymanager.com' 
WHERE OptionID = 'patientportal.host';

UPDATE tblConfigurationOption 
SET OptionValue = 'US' 
WHERE OptionID = 'locale.default_country';

ALTER TABLE tblConfiguration ADD Region varchar(5);

UPDATE tblConfiguration SET Region = 'US';
```

---

## Server Management

### Development Server URLs

- **SOMS (CTMS/edocs/eSource):** `dev.realtime-host01.com`
- **Enterprise:** `dev-ent.realtime-host01.com`

### Server IP Addresses

- **SOMS Web Dev:** `10.210.128.220`
- **ENT Web Dev:** `10.210.128.202`

### RDX Server Setup

1. **Initial setup:**

```bash
cd /home/abhishek
sudo chmod -R 600 *
```

1. **Switch to realtime user:**

```bash
sudo su realtime
```

1. **Clone repository:**

```bash
git clone -b main git@github.com:RealTime-Software-Solutions/ctms.git text161
exit
```

### SSH Configuration

```bash
# For SSH key issues
sudo chmod 700 .ssh
cd .ssh
ls -ltra
```

---

## File Transfer Commands

### Server to Local

```bash
scp abhishek@3.16.234.223:~/tmp_ctrg.sql .
```

### Local to Server

```bash
scp 'dbname' abhishek@<IP>:~/.
scp 'dbname' abhishek@<IP>:/srv/http/ctms/.
```

---

## Troubleshooting

### Login Issues

```bash
cd /srv/http/ctms/esrc706/sessions
sudo chmod -Rf 777 .
```

### Git Permission Issues

```bash
cd /.git
sudo chown -Rf abhishek:devops .
```

### Cron Permission Issues

```bash
cd rc17-engage  # instance directory
cd documents/
ls -lah
sudo chown -Rf realtime:devops ./*
ls -lah
```

---

## Git Operations

### Git Configuration

```bash
# Enable colored output
git config --global colour.ui true
```

### Branch Management

```bash
# Rename branch
git branch -m OLD-BRANCH-NAME NEW-BRANCH-NAME
git fetch origin
git branch -u origin/OLD-BRANCH-NAME NEW-BRANCH-NAME
git remote set-head origin -a
```

### Commit Message Correction

#### For Most Recent Commit

```bash
git commit --amend -m "Correct commit message"
git push --force-with-lease
```

#### For Older Commits

1. Start interactive rebase:

```bash
git rebase -i HEAD~n  # where n is number of commits back
```

1. Change "pick" to "reword" for the target commit
2. Edit the commit message in the new editor
3. Push changes:

```bash
git push --force-with-lease
```

### Git Stash Operations

**Basic stashing:**

```bash
git stash                    # Save current changes
git stash list              # View all stashes
git stash apply             # Apply most recent stash
git stash pop               # Apply and remove most recent stash
```

**Advanced stash operations:**

```bash
git stash show stash@{0}           # View stash details
git stash apply stash@{1}          # Apply specific stash
git stash drop stash@{0}           # Delete specific stash
git stash clear                    # Delete all stashes
```

### Automated Setup Scripts

**Create executable script:**

```bash
chmod +x setup_environment.sh
```

**Create input file for provisioning:**

```bash
# Create provision_input.txt with:
us-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com
3306
sbox
sbox
al_esrc1289
```

**Modify script to use input redirection:**

```bash
php util/provision.php < /srv/http/ctms/abhishek/provision_input.txt
```

---

## MCP Server Setup for Windsurf

### Server-Side Setup

1. **Connect to server:**

```bash
ssh username@<server-ip>
```

1. **Run initial setup:**

```bash
soms_dev_setup.sh setup
```

1. **Navigate to MCP folder:**

```bash
cd somsMCP/
```

1. **Configure environment:**

```bash
nano .env
```

```env
DB_HOST=us-soms-rds-dev.cdbscgbrzbgp.us-east-2.rds.amazonaws.com
DB_PORT=3306
DB_DATABASE=soms_mcp
DB_USERNAME=sbox
DB_PASSWORD=sbox
```

1. **Test database connection:**

```bash
php test_database_connection.php
```

1. **Start MCP instance:**

```bash
php <developername>_start.php  # Example: php tejas_start.php
```

### Windsurf Configuration

1. **Open Windsurf** → Cascade Customizations → MCP Servers → Manage Servers

2. **View Raw Config** and add:

```json
{
  "mcpServers": {
    "SOMS MCP - tejas": {
      "serverUrl": "http://10.210.128.31:9100/",
      "headers": {
        "Authorization": "Bearer 56b13ee8344d7fb5de001347159f6d6bb2b06b32d29172a644b62a3f81911ab1",
        "Content-Type": "application/json"
      }
    }
  }
}
```

1. **Save (Ctrl+S)** → **Refresh**

### Background Process Management

**Using tmux:**

```bash
tmux new -s mcp
php tejas_start.php
# Detach: Ctrl+B → D
# Reattach: tmux attach -t mcp
```

---

## Additional Notes

### Search Pull Requests by Commit ID

```
https://github.com/search?q=[commit-id]&type=pullrequests
```

### Server Access Credentials

**RDS Server Login:**

```bash
ssh shubhamsingh@10.210.128.220
# Password: 6Qf68~4M9.)N6?"O
```

### SSH Configuration Examples

```ssh-config
Host 10.210.128.220
  HostName 10.210.128.220
  User shubhamsingh

Host github-js-leetcode
    HostName github.com
    User git
    IdentityFile C:\Users\AbhishekLodh\.ssh\id_rsa_js_leetcode
```

---

*This documentation serves as a comprehensive guide for CTMS/SOMS development environment setup and daily operations. Keep it updated as processes evolve.*