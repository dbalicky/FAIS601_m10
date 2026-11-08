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