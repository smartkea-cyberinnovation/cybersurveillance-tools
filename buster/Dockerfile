# ────────────────────────────────────────────────────────────
# Imagen base: slim (ligera) pero con todo Python 3.10 LTS
FROM python:3.10-slim

# Evita archivos .pyc y permite ver los logs en tiempo real
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# — Dependencias de sistema —
# gcc + build-essential → compila extensiones C
# libxml2/libxslt → lxml
# libffi/libssl    → gevent/grequests, etc.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git build-essential gcc \
        libxml2-dev libxslt1-dev \
        libffi-dev  libssl-dev && \
    rm -rf /var/lib/apt/lists/*

# — Descarga del proyecto —
WORKDIR /opt
RUN git clone --depth 1 https://github.com/sham00n/buster.git
WORKDIR /opt/buster

# — Instalación en modo «editable» —
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir .

# Deja «buster» como comando por defecto
ENTRYPOINT ["buster"]
CMD ["-h"]

