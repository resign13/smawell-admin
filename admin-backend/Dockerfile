FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt gunicorn

COPY . .

EXPOSE 5302

CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5302", "app:app", "--timeout", "120"]
