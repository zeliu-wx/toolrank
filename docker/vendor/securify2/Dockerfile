FROM ubuntu:18.04
ARG SOLC=0.5.12

RUN sed -i 's|archive.ubuntu.com|mirrors.aliyun.com|g; s|security.ubuntu.com|mirrors.aliyun.com|g' /etc/apt/sources.list

# install basic packages
RUN apt-get update && apt-get install -y\
    software-properties-common\
    locales

# set correct locale
RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

# install tools
RUN apt-get update &&\
        apt-get -y install\
        wget\
        gdebi

# install souffle
RUN wget https://github.com/souffle-lang/souffle/releases/download/1.6.2/souffle_1.6.2-1_amd64.deb -O /tmp/souffle.deb &&\
        gdebi --n /tmp/souffle.deb

# install graphviz and python (plus build deps for pip packages)
RUN apt-get update && apt-get -y install\
        graphviz \
        python3.7 \
        python3.7-venv \
        python3.7-distutils \
        python3.7-dev \
        python3-pip \
        build-essential \
        ca-certificates \
        curl

# install the required solc version
RUN curl -L https://github.com/ethereum/solidity/releases/download/v$SOLC/solc-static-linux > /usr/bin/solc-$SOLC && \
    chmod +x /usr/bin/solc-$SOLC && \
    ln -s /usr/bin/solc-$SOLC /usr/local/bin/solc

COPY requirements.txt /requirements.txt

WORKDIR /sec

# copy and compile securify
COPY . /sec
ENV PYTHONPATH=/sec

# install securify requirements
RUN set -eux; \
    python3.7 -V; \
    (python3.7 -m ensurepip --upgrade --default-pip || true); \
    if ! python3.7 -m pip --version; then \
      curl -sS https://bootstrap.pypa.io/pip/3.7/get-pip.py -o /tmp/get-pip.py; \
      python3.7 /tmp/get-pip.py "pip<24.1" "setuptools<70" "wheel<0.43"; \
    else \
      python3.7 -m pip install --no-cache-dir --upgrade \
        "pip<24.1" \
        "setuptools<70" \
        "wheel<0.43"; \
    fi; \
    python3.7 -m pip --version; \
    python3.7 -m pip install --no-cache-dir -r /requirements.txt; \
    python3.7 -m pip install --no-cache-dir .

RUN cd /sec/securify/staticanalysis/libfunctors/ && ./compile_functors.sh

RUN cd /sec/securify/staticanalysis/souffle_analysis && \
        souffle --dl-program=../dl-program \
        --fact-dir=/sec/securify/staticanalysis/facts_in \
        --output-dir=/sec/securify/staticanalysis/facts_out \
        -L../libfunctors -w analysis.dl


ENV LD_LIBRARY_PATH=/sec/securify/staticanalysis/libfunctors



ENTRYPOINT ["python3.7", "securify/__main__.py"]
