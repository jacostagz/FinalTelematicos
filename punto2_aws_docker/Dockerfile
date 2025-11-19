FROM python:3.10-slim

WORKDIR /app

# Copiar la aplicación
COPY webapp /app/webapp
COPY ssl /app/ssl

# Instalar dependencias
RUN pip install flask flask_sqlalchemy gunicorn pymysql

# Exponer HTTPS
EXPOSE 443

# Ejecutar Gunicorn apuntando al módulo correcto
CMD ["gunicorn", "--certfile=/app/ssl/localhost.crt", "--keyfile=/app/ssl/localhost.key", "-b", "0.0.0.0:443", "webapp.web.views:app"]
