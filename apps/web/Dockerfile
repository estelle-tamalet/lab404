# syntax=docker/dockerfile:1

FROM dhi.io/python:3.14-debian13-dev@sha256:42fb1e613a3fd86e85c30f1b054536ba642c455123d6b4bcbd796aa49de4e46b AS build

ENV PYTHONDONTWRITEBYTECODE=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app
RUN python -m venv /app/venv
ENV PATH="/app/venv/bin:$PATH"

COPY requirements.lock .
RUN pip install --no-cache-dir --require-hashes -r requirements.lock \
 && python -m pip uninstall -y pip

FROM dhi.io/python:3.14-debian13@sha256:aa13af4b3777149bbf18dc6fe392c5ab9eb097d39715d57f4bacde537b6e0e26

ENV PATH="/app/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app
COPY --from=build /app/venv /app/venv
COPY . .

USER 65532:65532
EXPOSE 8080
ENTRYPOINT ["opentelemetry-instrument", "gunicorn", "--bind=0.0.0.0:8080", "--access-logfile=-", "app:app"]
