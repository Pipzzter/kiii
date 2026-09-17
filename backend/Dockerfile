FROM python:3.14-slim

ENV PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

COPY pyproject.toml alembic.ini ./
COPY app ./app

RUN pip install .

COPY scripts ./scripts

RUN mkdir -p app/frontend && \
    printf '<html><body>Backend API is running. UI is served by the frontend service.</body></html>' > app/frontend/index.html

EXPOSE 8000

CMD ["sh", "-c", "bash scripts/prestart.sh && fastapi run app/main.py --host 0.0.0.0 --port 8000"]
