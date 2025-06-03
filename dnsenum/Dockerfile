####################  Dockerfile  ####################
FROM debian:bookworm-slim

# --- Dependencias -----------------------------------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        dnsenum python3 ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# --- Script Python que PARSEA la salida de dnsenum --------------------------
RUN cat <<'PY' > /usr/local/bin/parse-dnsenum.py
#!/usr/bin/env python3
import sys, re, json, os

domain = os.getenv("DOMAIN_ENV", "")
raw    = sys.stdin.read()

hosts, nameservers, mailservers, records = [], [], [], []
current = None

# Patrones para detectar secciones
SECTIONS = {
    "hosts":       re.compile(r"Host'?s?\s+addresses?:", re.I),
    "nameservers": re.compile(r"Name\s+Servers?:",        re.I),
    "mailservers": re.compile(r"Mail.*\(MX\).*Servers?:", re.I),
    "hosts2":      re.compile(r"Brute forcing",           re.I),  # tratamos brute-force como hosts
}

for line in raw.splitlines():
    l = line.strip()

    # ¿Cambio de sección?
    for name, pat in SECTIONS.items():
        if pat.match(l):
            current = "hosts" if name == "hosts2" else name
            break
    else:
        # Saltar separadores y blancos
        if not l or l[0] in "_-":
            continue

        parts = l.split()
        if "IN" not in parts or len(parts) < 4:
            continue

        idx    = parts.index("IN")
        host   = parts[0].rstrip(".")
        ttl    = parts[1] if idx >= 2 else ""
        rtype  = parts[idx + 1]
        value  = " ".join(parts[idx + 2:]).rstrip(".")

        rec = {"hostname": host, "ttl": ttl, "type": rtype, "value": value}
        records.append(rec)

        if current == "hosts":
            hosts.append(rec)
        elif current == "nameservers":
            nameservers.append(rec)
        elif current == "mailservers":
            mailservers.append(rec)

json.dump({
    "domain":      domain,
    "hosts":       hosts,
    "nameservers": nameservers,
    "mailservers": mailservers,
    "records":     records
}, sys.stdout, indent=4)
PY

RUN chmod +x /usr/local/bin/parse-dnsenum.py

# --- Script de ENTRADA ------------------------------------------------------
RUN cat <<'BASH' > /usr/local/bin/dnsenum-json
#!/usr/bin/env bash
set -euo pipefail

if [ $# -lt 1 ]; then
    echo "Usage: $0 <domain> [extra dnsenum flags]" >&2
    exit 1
fi

DOMAIN="$1"; shift || true
OUT="${DOMAIN}-dnsenum.json"

# Ejecutar dnsenum y parsear su salida
dnsenum "$DOMAIN" \
        --threads 15 \
        --timeout 5 \
        --noreverse \
        --nocolor \
        -f /usr/share/dnsenum/dns.txt \
        "$@" 2>/dev/null | \
DOMAIN_ENV="$DOMAIN" /usr/local/bin/parse-dnsenum.py > "$OUT"

echo "✅ JSON generado en $(pwd)/$OUT"
BASH

RUN chmod +x /usr/local/bin/dnsenum-json

# Carpeta donde dejaremos los resultados
WORKDIR /data
ENTRYPOINT ["/usr/local/bin/dnsenum-json"]
#####################################################

