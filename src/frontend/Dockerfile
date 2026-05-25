FROM python:3.11-bookworm

WORKDIR /app

COPY . .

RUN pip install uv && uv pip install --system --no-cache .

CMD ["streamlit", "run", "app/main.py", "--server.port=8501", "--server.address=0.0.0.0"]
