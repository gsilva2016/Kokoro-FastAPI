FROM --platform=$BUILDPLATFORM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive

# Install Python and other dependencies
RUN apt-get update -y &&  \
    apt-get install -y python3.10 python3-venv espeak-ng espeak-ng-data git libsndfile1 curl ffmpeg g++ cmake && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    mkdir -p /usr/share/espeak-ng-data &&  \
    ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/ && \
    curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/ && \
    useradd -m -u 1001 appuser && \
    mkdir -p /app/api/src/models/v1_0 && \
    chown -R appuser:appuser /app

RUN apt update
RUN apt install -y ffmpeg vim wget curl sudo gpg git build-essential && rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*

RUN apt update; apt-get install -y software-properties-common; add-apt-repository -y ppa:kobuk-team/intel-graphics; apt-get install -y libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc; apt-get install -y intel-media-va-driver-non-free libmfx-gen1 libvpl2 libvpl-tools libva-glx2 va-driver-all vainfo; rm -rf /var/lib/apt/lists/*;rm -rf /tmp/*
RUN apt update; apt-get install -y libze-dev intel-ocloc && rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*

RUN usermod -a -G video appuser
#RUN usermod -a -G render appuser
RUN groupadd -g 110 render && usermod -aG 110 appuser


USER appuser
WORKDIR /app

# Copy dependency files
COPY --chown=appuser:appuser pyproject.toml ./pyproject.toml

# Install dependencies with GPU extras
RUN uv venv --python 3.10 && \
    uv sync --extra xpu --no-cache 

# Copy project files including models
COPY --chown=appuser:appuser api ./api
COPY --chown=appuser:appuser web ./web
COPY --chown=appuser:appuser docker/scripts/ ./
RUN chmod +x ./entrypoint.sh


# Set all environment variables in one go
ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    UV_LINK_MODE=copy \
    USE_GPU=true \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data \
    DEVICE="xpu"

ENV DOWNLOAD_MODEL=true
# Download model if enabled
RUN if [ "$DOWNLOAD_MODEL" = "true" ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

# Run FastAPI server through entrypoint.sh
CMD ["./entrypoint.sh"]
