FROM python:3.13-slim

ENV PIP_NO_CACHE_DIR=1 \
    PYTHONUNBUFFERED=1

COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

# Utente non privilegiato per gli studenti
RUN useradd --create-home --uid 1000 student
USER student
WORKDIR /home/student/notebooks

EXPOSE 8888

# Nessun token: il laboratorio gira solo in locale
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", \
     "--IdentityProvider.token=", "--ServerApp.password=", \
     "--ServerApp.root_dir=/home/student/notebooks"]
