# Jammy: This is the codename for Ubuntu 22.04 LTS. It’s the full version of Ubuntu 22.04 and is supported for five years from its release date.
# 2024-09-20
FROM ubuntu:22.04

# MySQL 8 shell + mysql client
RUN apt-get update -y
RUN apt-get install wget curl gnupg -y
RUN wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell_9.0.1-1ubuntu22.04_amd64.deb
RUN dpkg -i mysql-shell_9.0.1-1ubuntu22.04_amd64.deb
RUN apt-get install mysql-client -y
RUN rm mysql-shell_9.0.1-1ubuntu22.04_amd64.deb

# locale
RUN apt-get install locales -y && locale-gen "en_US.UTF-8" && update-locale LANG=en_US.UTF-8 && update-locale LANGUAGE=en_US.UTF-8 && update-locale LC_ALL=en_US.UTF-8

# install other utilities
RUN apt-get install unzip tree htop telnet inetutils-ping vim python3 tmux jq sudo bat -y

# Aws CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && ./aws/install
RUN rm awscliv2.zip

# Configure root
RUN echo "ubuntu ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/sudoers_custom
RUN echo "PS1='\[\033[0;31m\]\u@\h\[\033[00m\]:\[\033[01;94m\]\W\[\033[00m\]\# '" >> /root/.bashrc

# Create a new user 'ubuntu' with a home directory
RUN useradd -ms /bin/bash ubuntu
USER ubuntu
WORKDIR /home/ubuntu
RUN echo "PS1='\[\033[0;95m\]\u@\h\[\033[00m\]:\[\033[01;94m\]\W\[\033[00m\]\$ '" >> /home/ubuntu/.bashrc
RUN echo 'export LANG="en_US.UTF-8" \nexport LANGUAGE="en_US.UTF-8" \nexport LC_ALL="en_US.UTF-8"' >> /home/ubuntu/.bashrc
COPY RK_README.md .
CMD ["sleep", "infinity"]
