FROM python:3.9-slim

WORKDIR /app
COPY . /app/

RUN pip install --no-cache-dir flask

EXPOSE 8000

CMD ["python", "app.py"]

