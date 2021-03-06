FROM openjdk:jre-slim

ENV SONAR_SCANNER_MSBUILD_VERSION=4.1.0.1148 \
    SONAR_SCANNER_VERSION=3.1.0.1141 \
    SONAR_SCANNER_MSBUILD_HOME=/opt/sonar-scanner-msbuild \
    WHITESOURCE_HOME=/opt/whitesource \
    DOTNET_PROJECT_DIR=/project \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true \
    DOTNET_CLI_TELEMETRY_OPTOUT=true

RUN set -x \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    gnupg \
    dirmngr \
    curl \
    libunwind8 \
    gettext \
    apt-transport-https \
    wget \
  && curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg \
  && mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg \
  && sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-stretch-prod stretch main" > /etc/apt/sources.list.d/dotnetdev.list' \
  && apt-get update \
  && apt-get install dotnet-sdk-2.1.101 -y --no-install-recommends \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/SonarSource/sonar-scanner-msbuild/releases/download/$SONAR_SCANNER_MSBUILD_VERSION/sonar-scanner-msbuild-$SONAR_SCANNER_MSBUILD_VERSION-netcoreapp2.0.zip -O /opt/sonar-scanner-msbuild.zip \
  && mkdir -p $SONAR_SCANNER_MSBUILD_HOME \
  && mkdir -p $DOTNET_PROJECT_DIR \
  && unzip /opt/sonar-scanner-msbuild.zip -d $SONAR_SCANNER_MSBUILD_HOME \
  && rm /opt/sonar-scanner-msbuild.zip \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/*.dll \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/bin/* \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/lib/*.jar
  
RUN mkdir -p $WHITESOURCE_HOME
ADD https://s3.amazonaws.com/file-system-agent/whitesource-fs-agent-18.2.1.jar $WHITESOURCE_HOME
RUN chmod +x $WHITESOURCE_HOME/whitesource-fs-agent-18.2.1.jar
  
# Install Cloud Foundry cli
ADD https://cli.run.pivotal.io/stable?release=linux64-binary&version=6.32.0 /tmp/cf-cli.tgz
RUN mkdir -p /usr/local/bin && \
  tar -xzf /tmp/cf-cli.tgz -C /usr/local/bin && \
  cf --version && \
  rm -f /tmp/cf-cli.tgz

# Install cf cli Autopilot plugin
ADD https://github.com/contraband/autopilot/releases/download/0.0.3/autopilot-linux /tmp/autopilot-linux
RUN chmod +x /tmp/autopilot-linux && \
  cf install-plugin /tmp/autopilot-linux -f && \
  rm -f /tmp/autopilot-linux

# Install yaml cli
ADD https://github.com/mikefarah/yaml/releases/download/1.10/yaml_linux_amd64 /tmp/yaml_linux_amd64
RUN install /tmp/yaml_linux_amd64 /usr/local/bin/yaml && \
  yaml --help && \
  rm -f /tmp/yaml_linux_amd64

ENV PATH="$SONAR_SCANNER_MSBUILD_HOME:$SONAR_SCANNER_MSBUILD_HOME/sonar-scanner-$SONAR_SCANNER_VERSION/bin:${PATH}"
