# Mini Project Demo

[presentation-miniproject-en.pdf](https://github.com/user-attachments/files/33897024/presentation-miniproject-en.pdf)

# 1. Requirement Setting

#### **1. Environment Setting Edit**

> [!IMPORTANT]
> **If you wish to use the summarization feature, you must obtain and use an OpenAI API key.**

<details>
<summary>.env file</summary>

```env
# infra/.env.example

#### Require : Environment variables for the server services
WEB_FRONTEND_PORT=YOUR_FRONTEND_PORT

#### Require : MySQL environment variables for the project
MYSQL_DATABASE=YOUR_DATABASE_NAME
MYSQL_USER=YOUR_DATABASE_USER
MYSQL_PASSWORD=YOUR_DATABASE_PASSWORD
MYSQL_ROOT_PASSWORD=YOUR_DATABASE_ROOT_PASSWORD
MYSQL_HOST=YOUR_DATABASE_HOST
MYSQL_PORT=YOUR_DATABASE_PORT
MYSQL_CONTAINER_PORT=YOUR_DATABASE_CONTAINER_PORT

#### Require : Spring server environment variables
SPRING_SERVER_PORT=YOUR_SPRING_SERVER_PORT
SPRING_MYSQL_HOST=YOUR_DATABASE_HOST
# `SPRING_MYSQL_PORT` at Spring boot engine is default use lock configuration for MySQL connecting to `3306` port
SPRING_MYSQL_PORT=3306
SPRING_MYSQL_DATABASE=YOUR_DATABASE_NAME
SPRING_MYSQL_USER=YOUR_DATABASE_USER
SPRING_MYSQL_PASSWORD=YOUR_DATABASE_PASSWORD

JWT_SECRET=YOUR_JWT_SECRET

APP_OCR_BASE_URL=YOUR_OCR_BASE_URL
APP_OCR_CLIENT=YOUR_OCR_CLIENT

#### Require : Python server environment variables
PYTHON_SERVER_PORT=YOUR_PYTHON_SERVER_PORT
APP_NAME=YOUR_APP_NAME
APP_VERSION=YOUR_APP_VERSION
APP_ENV=YOUR_APP_ENV
DEBUG=YOUR_DEBUG_SETTING
AI_PROVIDER=YOUR_AI_PROVIDER

# If you wish to use the summarization feature, you must obtain and use an OpenAI API key.
OPENAI_API_KEY=YOUR_OPENAI_API_KEY
# Free OpenAI API plan is default selected to `gpt-6-luna`
OPENAI_MODEL=gpt-6-luna

PDF_BACKEND=PDFIUM
DEFAULT_OCR_ENGINE=TESSERACT

# We develop default timeout of 30 and max use token to 1000
AI_TIMEOUT_SECONDS=30
AI_MAX_OUTPUT_TOKENS=1000

NETWORK_ENV=OPEN
ENGINE=YOUR_ENGINE
```

</details>

#### **2. Web React Setting**

> [!WARNING]
> **Only don't host address to `localhost` is can't access in the docker container web page**

<details>
<summary>Vite Config Setting</summary>

```js
// ./web/vite.config.js
server: {
    host: 'YOUR_REACT_DEV_HOST',
    port: YOUR_REACT_DEV_PORT,
    strictPort: true,
    proxy: {
      '/api': {
        target: 'http://spring-server-backend:YOUR_SPRING_SERVER_PORT',
        changeOrigin: true,
      }
    },
  },
  preview: {
    host: 'YOUR_REACT_PREVIEW_HOST',
    port: YOUR_REACT_PREVIEW_PORT,
    strictPort: true,
    proxy: {
      '/api': {
        target: 'http://spring-server-backend:YOUR_SPRING_SERVER_PORT',
        changeOrigin: true,
      }
    },
  }
```

</details>

<details>
<summary>React Dockerfile</summary>

```dockefile
# ./web/Dockerfile
FROM node:24.21-slim

WORKDIR /app

COPY package.json /app
COPY package-lock.json /app

COPY . .

# Clean up apt cache to reduce image size
RUN echo "Installing dependencies..."
RUN npm install

RUN echo "Version check:"
RUN node --version && npm --version

RUN npm run build

EXPOSE YOUR_REACT_PORT

CMD ["npm", "run", "preview", "--", "--port", "YOUR_EXPOSE_REACT_PORT"]
```

</details>

#### **3. Spring boot Server Setting**

> [!IMPORTANT]
> 

<details>
<summary>Spring boot Setting Application</summary>

```yaml
# ./backend/src/main/resources/application.yaml
spring:
  application:
    name: backend
  datasource:
    url: jdbc:mysql://${MYSQL_HOST:YOUR_DATABASE_SERVICE_NAME}:${MYSQL_PORT:3306}/${MYSQL_DATABASE:YOUR_DATABASE_NAME}?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: ${MYSQL_USER:YOUR_DATABASE_USER}
    password: ${MYSQL_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate
    open-in-view: false
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 25MB
app:
  jwt:
    secret: ${JWT_SECRET}
    expiration-seconds: 3600
  storage:
    dir: ./storage
  ocr:
    client: mock
    base-url: http://0.0.0.0:${PYTHON_SERVER_PORT}
    timeout-seconds: 600
```

</details>

<details>
<summary>Spring boot Dockerfile</summary>

```dockefile
# ./backend/Dockerfile
# Build stage for the backend application
FROM eclipse-temurin:17-jdk AS build

WORKDIR /app

COPY . .

RUN echo "Access authorization for the Gradle wrapper of the backend application"
# Access authorization for the Gradle wrapper of [001 : --x]
RUN chmod +x gradlew

RUN echo "Making Gradle wrapper executable"
RUN ./gradlew bootJar --no-daemon

#---------------------------------------------------
# Multi-stage build: final image

FROM eclipse-temurin:17-jre

RUN echo "Setting working directory for the final image"
WORKDIR /app

RUN echo "Copying the built JAR from the build stage"
COPY --from=build /app/build/libs/*.jar app.jar

EXPOSE YOUR_SPRING_BOOT_SERVER_PORT

# ENTRYPOINT or CMD use is your want use option
# ENTRYPOINT ["java", "-jar", "app.jar", "--server.port=YOUR_SPRING_BOOT_SERVER_PORT"]
CMD ["java", "-jar", "app.jar", "--server.port=YOUR_SPRING_BOOT_SERVER_PORT"]
```

</details>

#### **4. Python Server Setting**

<details>
<summary>Python Dockerfile</summary>

```dockerfile
# ./ocr_api/Dockerfile
FROM python:3.12.15-slim

RUN echo "Updating apt and installing system dependencies..."
RUN apt-get update && apt-get install -y --no-install-recommends \
    tesseract-ocr tesseract-ocr-kor \
    libgl1 libglib2.0-0 libgomp1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

RUN echo "Copy requirements.txt to /app"
COPY requirements.txt .

COPY . .

RUN echo "Creating virtual environment at /opt/venv"
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN echo "Installing Python dependencies..."
RUN pip install --no-cache-dir -r requirements.txt

EXPOSE YOUR_PYTHON_FASTAPI_PORT

RUN echo "Starting the application..."
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "YOUR_PYTHON_FASTAPI_PORT"]
```

</details>

# 2. Create Docker Container

```bash
# ./infra/docker-compose.yml
docker compose up --build
```

# 3. Remove Docker Container

```bash
# ./infra/docker-compose.yml
docker compose down -v
```

# 4. Remove Docker Cache

```bash
# The this command is Docker in the have to remove cache clean(buffer disk)
docker builder prune -a

# Option : Docker for in the have all resource remove cache, build, image, volume, network, ...etc
docker system prune -a --volumes
```
