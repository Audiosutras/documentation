---
category: Docker
---

# Python Poetry Docker Examples

## Production

Example `Dockerfile`

```Dockerfile
FROM python:3.12-alpine

ARG POETRY_VERSION=1.7.1

ENV PYTHONFAULTHANDLER=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONHASHSEED=random
ENV PIP_NO_CACHE_DIR=off
ENV PIP_DISABLE_PIP_VERSION_CHECK=on
ENV PIP_DEFAULT_TIMEOUT=100

# System deps:
RUN apk add gcc musl-dev && \
pip install "poetry==$POETRY_VERSION"

# Copy only requirements to cache them in docker layer
WORKDIR /code
COPY poetry.lock pyproject.toml /code/

# No need to create a virtualenv for this isolated environment:
RUN poetry config virtualenvs.create false

# Creating folders, and files for a project:
COPY . /code
```

See [Github Action to Deploy Dockerfile to Remote Registry](/docs/docker_cd_github_action.md). Once the production Dockerfile is in place, upon a github release a new image will be created and added to the remote repository.

Example `compose.yaml` file.

```yaml
version: "3.8"

services:
  bot:
    # since this is production pull the latest image from ghcr.io or another remote registry
    image: ghcr.io/<username>/<repo_name>:latest
    # Here install the dependency groups that are need from pyproject.toml
    # Typically only the "main" dependencies in production
    command: sh -c "poetry install --no-interaction --only=main && <start command for project>"
    restart: always
    environment: # see .env_file if you rather specify a file instead of env vars individually
      # Add environment variables here
      - MONGODB_URI=mongodb://<MONGO_INITDB_ROOT_USERNAME>:<MONGO_INITDB_ROOT_PASSWORD>@mongo:27017/
    depends_on:
      - mongo
  # example of including a database service
  mongo:
    image: mongo
    restart: always
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=feed_bot
    volumes:
      - mongodata:/data/db
volumes:
  mongodata:
```

Within your production environment you will run the following to stand up your docker container.

```bash
docker compose up -d
```

## Development

Example `Dockerfile-dev`

```
FROM python:3.12-alpine

ARG POETRY_VERSION=1.7.1

ENV PYTHONFAULTHANDLER=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONHASHSEED=random
ENV PIP_NO_CACHE_DIR=off
ENV PIP_DISABLE_PIP_VERSION_CHECK=on
ENV PIP_DEFAULT_TIMEOUT=100

# System deps:
RUN apk add gcc musl-dev && \
pip install "poetry==$POETRY_VERSION"

# Copy only requirements to cache them in docker layer
WORKDIR /code
COPY poetry.lock pyproject.toml /code/

# No need to create a virtualenv for this isolated environment:
RUN poetry config virtualenvs.create false

# Run as a non-privileged user
RUN addgroup -S code && adduser -S -G code code --home /home/code --shell /bin/bash
RUN chown -R code:code /code
RUN chmod 755 /code
USER code

# Copy source files into application directory
COPY --chown=code:code . /code
```

Example docker compose file. This file could be `compose.yaml` and your production compose file could be `compose-prod.yaml` or some variation of this to distinguish between the files. Its up to you.

```yaml
version: "3.8"

services:
  bot:
    build:
      context: .
      dockerfile: Dockerfile-dev
    develop:
      ### provides hot reloading
      watch:
        - action: sync+restart
          # paths are relative to compose file
          path: ./<python app>
          target: /code/<python app>
        - action: rebuild
          path: pyproject.toml
    # Note: tests dependencies are included alongside main dependencies.
    command: sh -c "poetry install --no-interaction --only=main,tests && <start command for project>"
    restart: always
    environment:
      - MONGODB_URI=mongodb://<MONGO_INITDB_ROOT_USERNAME>:<MONGO_INITDB_ROOT_PASSWORD>@mongo:27017/
    depends_on:
      - mongo
  # example db service. In this case mongodb
  mongo:
    image: mongo
    restart: always
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=feed_bot
    volumes:
      - mongodata:/data/db
volumes:
  mongodata:
```

Run the project locally with `docker compose watch` to utilize hot reloading.
