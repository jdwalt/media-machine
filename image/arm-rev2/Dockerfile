FROM arm-qsv:2.24.3-hb1.9.2

LABEL org.opencontainers.image.title="ARM QSV with CRC identification guard Rev2"
LABEL org.opencontainers.image.version="2.24.3-hb1.9.2-ident-rev2-20260818"
LABEL local.arm.identify-revision="rev2"
LABEL local.arm.identify-sha256="f33fa61823fb7abfa05fad5b33355187aea34885d11a3bd3c70249e6ff69f14c"

COPY --chmod=0644 identify.py /opt/arm/arm/ripper/identify.py

RUN python3 -c \
    'path="/opt/arm/arm/ripper/identify.py"; compile(open(path, encoding="utf-8").read(), path, "exec")'
