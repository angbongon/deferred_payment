FROM python:3.9-slim-buster

ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV POETRY_VERSION=1.1.11

RUN apt-get update \
    # dependencies for building Python packages
    && apt-get install -y build-essential \
    # dependecy for downloading poetry
    && apt-get install -y curl \
    # psycopg2 dependencies
    && apt-get install -y libpq-dev \
    # Translations dependencies
    && apt-get install -y gettext \
    # cleaning up unused files
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/${POETRY_VERSION}/get-poetry.py | python - 

ENV PATH="${PATH}:/root/.poetry/bin"

# Requirements are installed here to ensure they will be cached.
COPY ./poetry.lock ./pyproject.toml ./

# All imports needed for autodoc.
RUN poetry config virtualenvs.create false \
    && poetry install --remove-untracked --no-interaction --no-ansi

# Copy content
COPY ./mkdocs.yml ./app/
COPY ./docs ./app/docs/

COPY ./compose/local/docs/start /start-docs
RUN sed -i 's/\r$//g' /start-docs
RUN chmod +x /start-docs

WORKDIR /app
