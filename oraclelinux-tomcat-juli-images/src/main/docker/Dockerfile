#
# Copyright 2025-2026 Open Text.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

ARG JAVA_VERSION

#
# Preliminary image that downloads Tomcat and makes some modifications
#
FROM ${projectDockerRegistry}/library/oraclelinux AS builder
RUN microdnf -y install gzip tar libxslt
WORKDIR /wd

ENV TOMCAT_PARENT_DIR=/wd/root/usr/share \
    TOMCAT_DIR_NAME=tomcat
ENV TOMCAT_ROOT_DIR $TOMCAT_PARENT_DIR/$TOMCAT_DIR_NAME
ENV TOMCAT_CONF_DIR $TOMCAT_ROOT_DIR/conf

# Download, extract Tomcat and remove unwanted web applications
RUN curl -fsSL -o /wd/apache-tomcat-11.0.27.tar.gz https://archive.apache.org/dist/tomcat/tomcat-11/v11.0.27/bin/apache-tomcat-11.0.27.tar.gz && \
    echo a617896307cada4884b24f80c4e977ddad8b3e0f88ce0e2a6eea29d5f332498e276ae5976c227a9c8daecf05c7c90def684a3766dd5a81f47a56d8b1adf43dc3 \
        /wd/apache-tomcat-11.0.27.tar.gz | sha512sum -c - && \
    mkdir -p $TOMCAT_PARENT_DIR && \
    cd $TOMCAT_PARENT_DIR && \
    tar xzf /wd/apache-tomcat-11.0.27.tar.gz && \
    mv apache-tomcat-11.0.27 $TOMCAT_DIR_NAME && \
    cd $TOMCAT_ROOT_DIR/webapps && \
    rm -rf docs/ examples/ host-manager/ manager/

# Overwrite the main logging configuration file
# (It is being copied to the working folder first, and then overwritten using cp, in order to retain the original file's permissions.)
COPY logging.properties .
RUN cp logging.properties $TOMCAT_CONF_DIR

# Apply the specified transform to the server.xml file
COPY server.xslt .
RUN cp $TOMCAT_CONF_DIR/server.xml .
RUN xsltproc -o $TOMCAT_CONF_DIR/server.xml server.xslt server.xml

# Add in the startup scripts
COPY startup.d/ root/startup/startup.d/
RUN chmod +x root/startup/startup.d/*

#
# The actual image definition
#
FROM ${projectDockerRegistry}/cafapi/oraclelinux-jre$JAVA_VERSION

RUN microdnf -y --refresh upgrade && \
    microdnf clean all

ENV CATALINA_HOME /usr/share/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
WORKDIR $CATALINA_HOME

HEALTHCHECK CMD curl --fail http://localhost:8081/healthcheck || exit 1

EXPOSE 8080 8081 8443
CMD ["catalina.sh", "run"]

COPY --from=builder /wd/root/ /
