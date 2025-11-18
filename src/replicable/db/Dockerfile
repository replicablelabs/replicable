FROM postgres:15-alpine

ARG POSTGRES_USER
ARG POSTGRES_PASSWORD
ARG POSTGRES_DB
ENV POSTGRES_USER=${POSTGRES_USER:-replicable}
ENV POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-replicablepwd}
ENV POSTGRES_DB=${POSTGRES_DB:-replicable}

## Install build deps and Python + Alembic dependencies
RUN apk add --no-cache python3 py3-pip build-base postgresql-dev

WORKDIR /app

# Copy dependency list first for better layer caching
COPY src/replicable/db/python-requirements.txt ./replicable/db/python-requirements.txt

# Install Python dependencies system-wide (using --break-system-packages due to PEP 668 on Alpine)
RUN python3 -m pip install --upgrade pip --break-system-packages \
 && pip install --no-cache-dir --break-system-packages -r ./replicable/db/python-requirements.txt \
 && rm -rf /root/.cache/pip

# Copy application modules needed for migrations
COPY src/replicable/alembic.ini ./alembic.ini
COPY src/replicable/alembic ./replicable/alembic
COPY src/replicable/core ./replicable/core
COPY src/replicable/db ./replicable/db
COPY src/replicable/models ./replicable/models
COPY src/replicable/schemas ./replicable/schemas
COPY src/replicable/repositories ./replicable/repositories
COPY src/replicable/services ./replicable/services
COPY src/replicable/__init__.py ./replicable/__init__.py

# Set PYTHONPATH so alembic env imports work
ENV PYTHONPATH=/app
RUN mkdir -p /docker-entrypoint-initdb.d /app/bin

COPY src/replicable/db/bin/run_with_migrations.sh /app/bin/run_with_migrations.sh
RUN chmod +x /app/bin/run_with_migrations.sh

# Expose the standard Postgres port
EXPOSE 5432

# Use wrapper that invokes original entrypoint then migrations (idempotent)
ENTRYPOINT ["/app/bin/run_with_migrations.sh"]
