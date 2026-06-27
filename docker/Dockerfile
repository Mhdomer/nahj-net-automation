FROM python:3.12-slim

RUN apt-get update && apt-get install -y openssh-client sshpass \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir \
    ansible \
    ansible-lint \
    paramiko \
    netmiko \
    && ansible-galaxy collection install cisco.ios

WORKDIR /project