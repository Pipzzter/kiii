FROM python:3.14-slim

ENV PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

COPY pyproject.toml alembic.ini ./
COPY app ./app

RUN pip install .

COPY scripts ./scripts

# The app also knows how to serve a bundled frontend from app/frontend, but
# here the React app runs in its own container, so leave a placeholder
# instead of pulling frontend build tooling into the backend image.
RUN mkdir -p app/frontend && \
    printf '<html><body>Backend API is running. UI is served by the frontend service.</body></html>' > app/frontend/index.html

EXPOSE 8000

CMD ["sh", "-c", "bash scripts/prestart.sh && fastapi run app/main.py --host 0.0.0.0 --port 8000"]
