FROM python:3.12-slim AS compile-image
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN apt-get update
RUN apt-get install -y --no-install-recommends \
    git \
    g++ \
    cmake \
    libssl-dev

RUN python -m venv /opt/venv

# Make sure we use the virtualenv:
ENV PATH="/opt/venv/bin:$PATH"

# duckdb
# we need to compile duckdb ourselves because duckdb doesnt provide
# binary extensions for 'httpfs' in platform: linux_arm64_gcc4
# this means duckdb is not working to query remote files in
# both Mac M1 (only under docker) and Linux ARM (only under docker)
# Note: without docker, duckdb extensions autoload mechanism works.
# More info at:
# https://github.com/duckdb/duckdb/issues/8035

RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
    git clone --depth 1 --branch v1.0.0 https://github.com/duckdb/duckdb && \
    cd duckdb/tools/pythonpkg && python -m pip install . ; \
    elif [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
    pip install duckdb==1.0.0; \
    fi

FROM python:3.12-slim AS build-image

COPY --from=compile-image /opt/venv /opt/venv

# Make sure we use the virtualenv:
ENV PATH="/opt/venv/bin:$PATH"

COPY dagster-dbt dagster-dbt

WORKDIR /dagster-dbt

RUN pip install -e ".[dev]"

WORKDIR /dagster-dbt/postcard_company_dm/dbt

RUN dbt deps

WORKDIR /dagster-dbt/postcard_company_dm
