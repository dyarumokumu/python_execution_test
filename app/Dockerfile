# syntax=docker/dockerfile:1
ARG APP_NAME="app" # アプリ名
ARG BASE_IMAGE='nvidia/cuda' # 大元となるnvidia-driverの入ったdocker image名
ARG BASE_IMAGE_TAG='12.0.1-cudnn8-runtime-ubuntu20.04'
ARG PYTHON_VERSION='3.9' # pythonのバージョン
ARG PYTHON_BASE_IMAGE='python' # 実行時のベースイメージ
ARG POETRY_CACHE_DIR='/root/.cache/poetry' # poetryのキャッシュディレクトリ

FROM ${BASE_IMAGE}:${BASE_IMAGE_TAG} AS poetry
ARG PYTHON_VERSION
RUN apt-get update \
    && DEBIAN_FRONTEND="noninteractive" apt-get install -y python${PYTHON_VERSION} python3-pip \
    && echo "alias python='python${PYTHON_VERSION}'" >> ~/.bashrc
ENV PYTHONUNBUFFERED=true
RUN \
    --mount=type=cache,target=/var/lib/apt/lists \
    --mount=type=cache,target=/var/cache/apt/archives \
    apt-get update \
    && apt-get install -y --no-install-recommends build-essential

RUN \
    --mount=type=cache,target=/root/.cache/pip \
    pip install -U pip poetry

FROM poetry AS dev
ARG APP_NAME
ARG POETRY_CACHE_DIR
WORKDIR /workspace/${APP_NAME}
COPY pyproject.toml poetry.lock ./
RUN apt-get update \
    && apt-get install -y git
RUN \
    --mount=type=cache,target=${POETRY_CACHE_DIR} \
    poetry config cache-dir ${POETRY_CACHE_DIR} && \
    poetry config installer.parallel false && \
    poetry config virtualenvs.create false && \
    poetry install --no-interaction

FROM poetry AS run
ARG POETRY_CACHE_DIR
WORKDIR /run
COPY pyproject.toml poetry.lock ./
RUN \
    --mount=type=cache,target=${POETRY_CACHE_DIR} \
    poetry config virtualenvs.in-project true && \
    poetry install --no-interaction
COPY . /run/${APP_NAME}
WORKDIR /run/${APP_NAME}
ENV APP_NAME=${APP_NAME}
ENTRYPOINT python -m ${APP_NAME}
