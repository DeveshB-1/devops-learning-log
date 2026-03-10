# Python DevOps Notes

## PyTest REST API Testing Framework

### Basic Structure

```python
# conftest.py
import pytest
import requests

BASE_URL = "https://api.example.com"

@pytest.fixture(scope="session")
def api_session():
    session = requests.Session()
    # Login / get token
    resp = session.post(f"{BASE_URL}/auth/login", json={
        "username": "admin",
        "password": "secret"
    })
    token = resp.json()["token"]
    session.headers.update({"Authorization": f"Bearer {token}"})
    yield session
    session.close()

@pytest.fixture
def base_url():
    return BASE_URL
```

```python
# test_users_api.py
import pytest

def test_get_users(api_session, base_url):
    resp = api_session.get(f"{base_url}/users")
    assert resp.status_code == 200
    data = resp.json()
    assert isinstance(data, list)
    assert len(data) > 0

def test_create_user(api_session, base_url):
    payload = {"name": "Test User", "email": "test@example.com"}
    resp = api_session.post(f"{base_url}/users", json=payload)
    assert resp.status_code == 201
    assert resp.json()["email"] == payload["email"]

def test_update_user(api_session, base_url):
    user_id = 1
    resp = api_session.put(f"{base_url}/users/{user_id}", json={"name": "Updated"})
    assert resp.status_code == 200
    assert resp.json()["name"] == "Updated"

@pytest.mark.parametrize("endpoint,expected", [
    ("/health", 200),
    ("/metrics", 200),
    ("/nonexistent", 404),
])
def test_endpoints(api_session, base_url, endpoint, expected):
    resp = api_session.get(f"{base_url}{endpoint}")
    assert resp.status_code == expected
```

```bash
# Run tests
pytest tests/ -v
pytest tests/ -v --tb=short -q        # quiet mode
pytest tests/ -k "test_users"         # filter by name
pytest tests/ -m "smoke"              # run smoke tests
pytest tests/ --cov=src --cov-report=html  # coverage report
```

## System Automation Scripts

### Process Monitor

```python
import psutil
import smtplib
from email.mime.text import MIMEText
import time

THRESHOLDS = {
    'cpu': 85.0,
    'memory': 85.0,
    'disk': 85.0
}

def check_system():
    alerts = []
    
    cpu = psutil.cpu_percent(interval=1)
    if cpu > THRESHOLDS['cpu']:
        alerts.append(f"HIGH CPU: {cpu:.1f}%")
    
    mem = psutil.virtual_memory()
    if mem.percent > THRESHOLDS['memory']:
        alerts.append(f"HIGH MEMORY: {mem.percent:.1f}%")
    
    disk = psutil.disk_usage('/')
    if disk.percent > THRESHOLDS['disk']:
        alerts.append(f"HIGH DISK: {disk.percent:.1f}%")
    
    return alerts

def send_alert(message):
    msg = MIMEText(message)
    msg['Subject'] = 'System Alert'
    msg['From'] = 'monitor@server.local'
    msg['To'] = 'admin@example.com'
    # smtp config here

if __name__ == "__main__":
    while True:
        alerts = check_system()
        if alerts:
            send_alert("\n".join(alerts))
            print("ALERTS:", alerts)
        time.sleep(60)
```

### Parallel API Requests

```python
import asyncio
import aiohttp
from typing import List, Dict

async def fetch(session: aiohttp.ClientSession, url: str) -> Dict:
    async with session.get(url) as resp:
        return {"url": url, "status": resp.status, "data": await resp.json()}

async def fetch_all(urls: List[str]) -> List[Dict]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)

# Usage
urls = [f"https://api.example.com/item/{i}" for i in range(1, 101)]
results = asyncio.run(fetch_all(urls))
```

## Useful Python Snippets for DevOps

```python
import subprocess, json, os

# Run shell command
def run(cmd, check=True):
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    if check and result.returncode != 0:
        raise RuntimeError(f"Command failed: {result.stderr}")
    return result.stdout.strip()

# Parse JSON config
def load_config(path):
    with open(path) as f:
        return json.load(f)

# Retry decorator
from functools import wraps
import time

def retry(times=3, delay=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == times - 1:
                        raise
                    print(f"Attempt {attempt+1} failed: {e}. Retrying...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(times=3, delay=5)
def call_flaky_api(url):
    import requests
    return requests.get(url, timeout=10)
```
