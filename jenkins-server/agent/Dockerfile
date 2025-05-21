FROM jenkins/inbound-agent:latest

USER root
RUN apt-get update && apt-get install -y \
    curl git unzip python3-pip nodejs npm openjdk-17-jdk docker.io \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sSLo /tmp/sonar.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip \
    && unzip /tmp/sonar.zip -d /opt \
    && mv /opt/sonar-scanner-* /opt/sonar-scanner \
    && ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner
