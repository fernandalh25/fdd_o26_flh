FROM python:3.14.7-slim-bookworm
WORKDIR /app
COPY requirements.txt .
COPY app.py .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
