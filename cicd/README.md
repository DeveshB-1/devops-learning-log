# CI/CD Pipeline Notes

## GitLab CI/CD

### Pipeline Structure

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - deploy

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  DEPLOY_ENV: production

# Reusable anchor
.python_template: &python_template
  image: python:3.11-slim
  before_script:
    - pip install -r requirements.txt

lint:
  <<: *python_template
  stage: lint
  script:
    - flake8 src/
    - black --check src/

test:
  <<: *python_template
  stage: test
  script:
    - pytest tests/ -v --cov=src --cov-report=xml
  coverage: '/TOTAL.*\s+(\d+%)$/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main

deploy:
  stage: deploy
  script:
    - kubectl set image deploy/web web=$IMAGE_TAG
    - kubectl rollout status deploy/web
  environment:
    name: production
  only:
    - main
  when: manual
```

### Multi-environment Pipeline

```yaml
deploy_staging:
  stage: deploy
  script:
    - helm upgrade --install app-staging ./helm -f values-staging.yaml
  environment:
    name: staging
  only:
    - develop

deploy_prod:
  stage: deploy
  script:
    - helm upgrade --install app ./helm -f values-prod.yaml
  environment:
    name: production
  only:
    - main
  when: manual  # require manual approval
```

## Jenkins Declarative Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "myapp:${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/org/repo.git'
            }
        }
        
        stage('Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest tests/ -v --junitxml=test-results.xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }
        
        stage('Build Docker') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh "kubectl apply -f k8s/"
                sh "kubectl rollout status deploy/app"
            }
        }
    }
    
    post {
        failure {
            emailext(
                subject: "Pipeline FAILED: ${env.JOB_NAME}",
                body: "Build ${env.BUILD_URL} failed.",
                to: "devops@example.com"
            )
        }
    }
}
```

## GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          
      - name: Install & Test
        run: |
          pip install -r requirements.txt
          pytest tests/ -v

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Build & Push Docker
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

## Quality Gates

- **Lint**: flake8, shellcheck, hadolint (Dockerfile)
- **Test**: pytest with coverage threshold (>80%)
- **Security**: trivy (container scan), bandit (Python)
- **Build**: only on passing tests
- **Deploy**: manual approval for production
