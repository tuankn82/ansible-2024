FROM mcr.microsoft.com/devcontainers/python:1-3.12-bookworm

# OCI labels
LABEL org.opencontainers.image.source="https://github.com/tuankn82/ansible-2024"
LABEL org.opencontainers.image.description="Ansible-2024"
LABEL org.opencontainers.image.licenses="GPL-3.0"

# install poetry for python dependencies
RUN python3 -m pip install poetry~=1.8.2

# install useful system packages
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y sshpass vim iputils-ping
RUN apt-get clean

# install vault cli
ARG VAULT_VERSION=1.16.2
RUN curl -Lo /tmp/vault.zip \
    https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip \
    && cd /usr/local/bin \
    && unzip /tmp/vault.zip \
    && rm -f /tmp/vault.zip

# make prompt multiline cause it's too long by default
RUN sed -i -E -e '/PS1="\$/c\    PS1=" ${userpart} ${lightblue}\\w ${gitbranch}${removecolor}\\n\\$ "' ~/.bashrc

# enable bash completion for git
RUN echo "source /usr/share/bash-completion/completions/git" >> ~/.bashrc

# add hashicorp vault config
RUN echo 'export VAULT_ADDR="http://vault.home.tkn:8200"' >> ~/.bashrc
RUN echo 'export VAULT_SKIP_VERIFY=1' >> ~/.bashrc

WORKDIR /workspaces/ansible-2024

# set up poetry environment
COPY poetry.lock pyproject.toml ./
RUN poetry install

# symlink poetry env to ~/venv
RUN VN=$(poetry env list | head -n 1 | cut -d ' ' -f 1) \
    && ln -s ~/.cache/pypoetry/virtualenvs/$VN /venv

# install collection (to get dependencies)
ENV COLLECTION=collections/ansible/tuankn

COPY $COLLECTION/galaxy.yml .
RUN poetry run -- ansible-galaxy collection install .

# remove empty collection directory so it finds the one in our git repo
RUN rm -rf ~/.ansible/$COLLECTION

CMD ["/bin/bash", "-c", "SHELL=/bin/bash poetry shell"]