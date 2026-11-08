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

## Docker Deployment

### Rebuild up to date image if necessary
```bash
docker compose up -d --build
```


# Testing

## Test with pgAdmin and PostreSQL

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

## Test with Pytest

**Ran test to show detailed results for all code not covered**
```bash
pytest --cov=app --cov-report=term-missing -v
```

# Issues

## Integration test failures

### Value errors for test_user_auth tests
```bash
FAILED tests/integration/test_user_auth.py::test_password_hashing - ValueError: password cannot be longer than 72 bytes, truncate manually if necessary (e.g. my_password[:72])
...
...
```

**Update bcrypt version for better compatibility with passlib**

```bash
pip install bcrypt==4.0.1
```

### Run pytest for integration tests
```bash
pytest tests/integration/ -v
```

### Update bcrypt version in requirements.txt
```txt
bcrypt==4.0.1
```

## E2E test failures

**Getting following errors with test_e2e.py**
```bash
ERROR tests/e2e/test_e2e.py::test_hello_world - tests.conftest.ServerStartupError: Failed to start test server
ERROR tests/e2e/test_e2e.py::test_calculator_add - tests.conftest.ServerStartupError: Failed to start test server
ERROR tests/e2e/test_e2e.py::test_calculator_divide_by_zero - tests.conftest.ServerStartupError: Failed to start test server
```

### Check currently running containers
```bash
docker compose ps
```

**fastapi_calculator container was unhealthy**

### Stop unhealthy container
```bash
docker compose stop web
```

### Fix code in main.py

**Replaced:**
```python
@app.get("/")
async def read_root(request: Request):
    """
    Serve the index.html template.
    """
    return templates.TemplateResponse(
        request=request,
        name="index.html"
    )
```

## Assertion Errors

**Got following errors after running pytest --cov=app --cov-report=term-missing -v**

### Replace with fixed code in test_e2e.py

**Added expect import from playwright**
```python
from playwright.sync_api import expect
```

**Fixed expected result line in test_calculator_add**
```python
expect(page.locator('#result')).to_have_text('Result: 15')
```

**Fixed expected result line in test_calculator_divide_by_zero**
```python
expect(page.locator('#result')).to_have_text('Error: Cannot divide by zero!')
```

## Trivy Security Issues

### Fixing high severity issues with dependency version

**Added and updated dependency versions for dnspython and setuptools in requirements.txt**
```txt
dnspython==2.9.0
setuptools>=78.1.1
```

**Added correct dependency version installs in Dockerfile**
```Dockerfile
RUN apt-get update && \
   ...
   python -m pip install --no-cache-dir --upgrade pip "setuptools>=78.1.1" wheel && \
   python -m pip install --no-cache-dir -r requirements.txt && \
   ...
```

### IMPORTANT: COPY requirements.txt BEFORE installing dependencies
```Dockerfile
COPY requirements.txt .
```

### Upgrade Python version to 3.11

**Changed version in Dockerfile**
```Dockerfile
FROM python:3.11-slim
```

**Changed version in .github/workflows/test.yml**
```yml
jobs:
  test:
    ...
    steps:
      ...
        with:
          python-version: '3.11'
```

### Rebuild and Test

**Rebuilt image without using cache**
```bash
docker compose build --no-cache web
```

**Recreate FASTAPI container**
```bash
docker compose up -d --no-deps --force-recreate web
```

**Tested for updated python and dependency versions**
```bash
docker compose exec web python --version

docker compose exec web python -m pip show dnspython setuptools
```

## Github Actions issue: ModuleNotFoundError: No module named 'psycopg'

### Updated DATABASE_URLs to include psycopg2

**Updated line in app/config.py**
```python
DATABASE_URL: str = "postgresql+psycopg2://postgres:postgres@localhost:5432/fastapi_db"
```

**Updated line in .github/workflows/test.yml**
```yml
DATABASE_URL: str = "postgresql+psycopg2://postgres:postgres@localhost:5432/fastapi_db"
```

**Updated lines in docker-compose.yml**
```yml
DATABASE_URL: str = "postgresql+psycopg2://postgres:postgres@localhost:5432/fastapi_db"
```

## Fixing issue with Trivy security not detecting up to date setuptools version

### Add dependency verification to test.yml
```yml
- name: Verify setuptools version
  run: |
    docker run --rm --entrypoint python app:test -m pip show setuptools
    docker run --rm --entrypoint sh app:test -c 'find /usr/local/lib -name "setuptools*.dist-info"'
```

### Create .dockerignore file

**Added to .dockerignore**
```txt
venv/
.venv/
__pycache__/
*.pyc
.pytest_cache/
.git/
htmlcov/
.coverage
```

### Add skip-files line to test.yml
```yml
name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          ...
          skip-files: '**/pip/_vendor/bom.cdx.json'
```