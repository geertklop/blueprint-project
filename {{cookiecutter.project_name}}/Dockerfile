# --- BASE IMAGE WITH GENERAL SETTINGS --- #
FROM python:3.10-slim-buster AS base

# Important settings
ENV POETRY_VERSION=1.1.11 \
    POETRY_VIRTUALENVS_CREATE=false \
    POETRY_HOME=/opt/poetry \
    VENV_PATH=/opt/venv \
    # Nice-to-have optimizations
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on \
    PIP_DEFAULT_TIMEOUT=100

# Set path for using poetry and virtual env
ENV PATH="$VENV_PATH/bin:$POETRY_HOME/bin:$PATH"

## Install System Runtime dependencies
# OPENJDK-8 (for databricks-connect)
RUN apt-get update && apt-get install -y software-properties-common \
    && apt-add-repository 'deb http://security.debian.org/debian-security stretch/updates main' \
    && apt-get update \
    && apt-get install -y openjdk-8-jre

# ODBC Drivers
RUN apt-get install -y curl gnupg2 \
    && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y msodbcsql17 \
    && apt-get install -y unixodbc-dev

# Cleanup
RUN apt-get purge --auto-remove -y software-properties-common curl \
    && rm -rf /var/lib/apt/lists/* /var/tmp* /var/share/{doc*,man*} /var/cache/apt/* \
    && apt-get -qyy clean


# --- BUILD IMAGE WITH RUNTIME PYTHON DEPENDENCIES --- #
FROM base AS build

# Install Poetry
RUN apt-get update && apt-get install --no-install-recommends -y curl \
    && curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python \
    && apt-get purge --auto-remove -y curl \
    && rm -rf /var/lib/apt/lists/*

# Create virtual env (already activated in PATH)
RUN python -m venv $VENV_PATH

# Install Python runtime dependencies
WORKDIR /app
COPY pyproject.toml poetry.lock ./

RUN apt-get update && apt-get install -y gcc g++\
    && python -m pip install --upgrade pip \
    && poetry install --no-root --no-dev \
    && apt-get purge --auto-remove -y  gcc g++ \
    && rm -rf /var/lib/apt/lists/*

# --- DEV IMAGE WITH ADDITIONAL DEPENDENCIES --- #
FROM build as dev

# Install dev dependencies
RUN poetry install --no-root

# Copy all tests, scripts, settings, etc. and do editable install
COPY . .
RUN poetry install

CMD [ "python", "-m", "{{ cookiecutter.project_name_short }}.main" ]

# --- BUILD IMAGE WITH LOCAL PACKAGE --- #
FROM build AS build-full

# Copy local package and install it
COPY src/ src/
RUN poetry build --format wheel \
    && pip install dist/*.whl

# --- MINIMAL RELEASE IMAGE --- #
FROM base as release

# Copy pre-built app and dependencies from build image
COPY --from=build-full $VENV_PATH $VENV_PATH

WORKDIR /app
COPY metadata/ metadata/

CMD [ "python", "-m", "{{ cookiecutter.project_name_short }}.main" ]
