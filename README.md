# Setup 

## Initial Git repo Setup

### Set python version to 3.10.11
```bash
pyenv local 3.10.11
```

### Create and activate venv
```bash
python3 -m venv venv
source venv/bin/activate
```

### Install dependencies from requirements.txt
```bash
pip install -r requirements.txt
```

### Upgrade pip
```bash
pip install --upgrade pip
```
### Initialize and connect to GitHub
```bash
git init

git remote set-url origin git@github.com:dbalicky/FAIS601_m10.git
# or git remote add origin git@github.com:dbalicky/FAIS601_m10.git
```

### Check remote github repo

**Checked to see if correct github repo is listed**
```bash
git remote -v
```

### Initial Commit and Push

**Added and commited all files/folders as initial commit, then pushed to github**
```bash
git add .

git commit -m 'initial commit'

git push -u origin main
```

## Docker Setup

### Create and Setup Docker Repository

**Generated personal access tokens **

**Added username and token as secrets to GitHub repo**

**Updated tags in .gtibhu/workflows/test,yml with docker repo**
```yml
deploy:
    ...
    steps:
      ...
        with:
          push: true
          tags: |
            dbal7/fais601_m10:latest
            dbal7/fais601_m10:${{ github.sha }}
          ...
          cache-from: type=registry,ref=dbal7/fais601_m10:cache
          ...
```

# Testing with pgAdmin and PostreSQL

## Accessing pgAdmin

### Compose the image

**Terminated any running images**
```bash
docker compose down
```

**Composed the image**
```bash
docker compose up -d
```

### Open localhost:5050

*Used following command to check running images and names*
```bash 
docker compose ps
``` 
### Logged in to pgAdmin

**Username:** admin@example.com

**Password:** admin

### Confirm PostreSQL Server Accessibility

**Added server with following configuration**

General:
- Name: Module 10

Connection:
- Host: db
- Port: 5432
- Database: fastapi_db
- Username: postgres
- Password: postgres

**Checked under Servers -> Module 10 -> Databases to confirm database exists**