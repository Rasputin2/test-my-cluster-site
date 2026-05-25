FROM python:3.11-bookworm

WORKDIR /app

COPY . .

RUN pip install uv && uv pip install --system --no-cache .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
