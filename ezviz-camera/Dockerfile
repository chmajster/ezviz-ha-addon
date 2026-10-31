ARG BUILD_FROM=ghcr.io/home-assistant/aarch64-base:3.19
FROM $BUILD_FROM

# Retry temporary repository failures before trying a second HTTPS mirror.
RUN set -eu; \
    install_packages() { \
        for attempt in 1 2 3; do \
            if apk add --no-cache --timeout 30 "$@" \
                python3 py3-pip ffmpeg dos2unix; then \
                return 0; \
            fi; \
            if [ "$attempt" -lt 3 ]; then \
                echo "Alpine package installation failed (attempt ${attempt}/3); retrying in 5s..." >&2; \
                sleep 5; \
            fi; \
        done; \
        return 1; \
    }; \
    if ! install_packages; then \
        echo "Retrying Alpine package installation via dl-4.alpinelinux.org..." >&2; \
        sed 's#https://dl-cdn\.alpinelinux\.org/alpine/#https://dl-4.alpinelinux.org/alpine/#g' \
            /etc/apk/repositories > /tmp/ezviz-apk-repositories; \
        if ! install_packages --repositories-file /tmp/ezviz-apk-repositories; then \
            echo "Alpine package installation failed on both mirrors. Check the Home Assistant host's DNS and outbound HTTPS access, then retry installation. See the apk errors above." >&2; \
            exit 1; \
        fi; \
        rm -f /tmp/ezviz-apk-repositories; \
    fi

# Set working directory
WORKDIR /app

# Create virtual environment and install Python packages
ENV VIRTUAL_ENV=/app/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Copy requirements and install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY ezviz_stream.py .
COPY stream_to_pipe.py .
COPY http_server.py .
COPY stream_manager.py .
COPY run.sh .

# Fix line endings and make run script executable
RUN dos2unix run.sh && chmod a+x run.sh

# Run the application
CMD [ "/app/run.sh" ]
