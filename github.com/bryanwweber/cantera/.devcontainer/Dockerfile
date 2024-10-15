FROM python:3.12-bookworm
ARG NINJA_VERSION=1.12.1

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
COPY requirements.txt /requirements.txt
ENV DEBIAN_FRONTEND=noninteractive

COPY CMakeLists.txt /build/
WORKDIR /build/
RUN --mount=type=cache,target=/cache \
    --mount=target=/var/lib/apt/lists,type=cache,sharing=locked \
    --mount=target=/var/cache/apt,type=cache,sharing=locked \
    rm -f /etc/apt/apt.conf.d/docker-clean \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
    doxygen graphviz clang-16 gfortran libopenblas-openmp-dev ninja-build cmake \
    && ninja --version \
    && cmake -G Ninja -S . -B build \
    && cd build \
    && ninja \
    && cd .. \
    && rm -rf build

RUN --mount=type=cache,target=/root/cache \ 
    uv pip install --system -r /requirements.txt