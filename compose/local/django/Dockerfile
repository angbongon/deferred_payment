ARG PYTHON_VERSION=3.9-slim-buster

# define an alias for the specfic python version used in this file.
FROM python:${PYTHON_VERSION} as python

# Python build stage
FROM python as python-build-stage

ARG BUILD_ENVIRONMENT=local

# Install apt packages
RUN apt-get update && apt-get install --no-install-recommends -y \
  # dependencies for building Python packages
  build-essential \
  # psycopg2 dependencies
  libpq-dev

# Create Python Dependency and Sub-Dependency Wheels.
# RUN pip wheel --wheel-dir /usr/src/app/wheels  \
#  -r ${BUILD_ENVIRONMENT}.txt

# Python 'run' stage
FROM python as python-run-stage

ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV POETRY_VERSION=1.1.11
ENV BUILD_ENV ${BUILD_ENVIRONMENT}

WORKDIR ${APP_HOME}

# Install required system dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
  curl \
  # psycopg2 dependencies
  libpq-dev \
  # Translations dependencies
  gettext \
  git \
  # cleaning up unused files
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip

# All absolute dir copies ignore workdir instruction. All relative dir copies are wrt to the workdir instruction
# copy python dependency wheels from python-build-stage
# COPY --from=python-build-stage /usr/src/app/wheels  /wheels/

RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/${POETRY_VERSION}/get-poetry.py | python -

ENV PATH="${PATH}:/root/.poetry/bin"

# Requirements are installed here to ensure they will be cached.
COPY ./poetry.lock ./pyproject.toml ./

RUN poetry config virtualenvs.create false \
  && poetry install --remove-untracked --no-interaction --no-ansi

# use wheels to install python dependencies
# RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
#	  && rm -rf /wheels/

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint

COPY ./compose/local/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start


COPY ./compose/local/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r$//g' /start-celeryworker
RUN chmod +x /start-celeryworker

COPY ./compose/local/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r$//g' /start-celerybeat
RUN chmod +x /start-celerybeat

COPY ./compose/local/django/celery/flower/start /start-flower
RUN sed -i 's/\r$//g' /start-flower
RUN chmod +x /start-flower


# copy application code to WORKDIR
COPY . ${APP_HOME}

ENTRYPOINT ["/entrypoint"]
