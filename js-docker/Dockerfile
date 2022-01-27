# Copyright (c) 2020. TIBCO Software Inc.
# This file is subject to the license terms contained
# in the license file that is distributed with this file.

ARG TOMCAT_BASE_IMAGE=tomcat:9.0.54-jdk11-openjdk
FROM ${TOMCAT_BASE_IMAGE}

ARG DN_HOSTNAME
ARG KS_PASSWORD
ARG JRS_HTTPS_ONLY
ARG HTTP_PORT
ARG HTTPS_PORT
ARG POSTGRES_JDBC_DRIVER_VERSION
ARG JASPERREPORTS_SERVER_VERSION
ARG EXPLODED_INSTALLER_DIRECTORY
ARG SMTP_HOST
ARG SMTP_PORT
ARG SMTP_PROTOCOL
ARG SMTP_EMAIL
ARG SMTP_USER
ARG SMTP_PASSWORD

ENV JRS_HOME /usr/src/jasperreports-server
ENV PHANTOMJS_VERSION ${PHANTOMJS_VERSION:-2.1.1}
ENV DN_HOSTNAME ${DN_HOSTNAME:-localhost.localdomain}
ENV KS_PASSWORD ${KS_PASSWORD:-changeit}
ENV JRS_HTTPS_ONLY ${JRS_HTTPS_ONLY:-false}
ENV HTTP_PORT ${HTTP_PORT:-8080}
ENV HTTPS_PORT ${HTTPS_PORT:-8443}
ENV POSTGRES_JDBC_DRIVER_VERSION ${POSTGRES_JDBC_DRIVER_VERSION:-42.3.1}
ENV JASPERREPORTS_SERVER_VERSION ${JASPERREPORTS_SERVER_VERSION:-8.0.0}
ENV EXPLODED_INSTALLER_DIRECTORY ${EXPLODED_INSTALLER_DIRECTORY:-resources/jasperreports-server-cp-$JASPERREPORTS_SERVER_VERSION-bin}
ENV JASPERREPORTS_EMAIL ${JASPERREPORTS_EMAIL:-noreply@jasperserver.tld}
ENV SMTP_HOST ${SMTP_HOST:-smtp.mailtrap.io}
ENV SMTP_PORT ${SMTP_PORT:-587}
ENV SMTP_PROTOCOL ${SMTP_PROTOCOL:-smtp}
ENV SMTP_EMAIL ${SMTP_EMAIL:-noreply@jasperserver.tld}
ENV SMTP_USER ${SMTP_USER:-mail_user}
ENV SMTP_PASSWORD ${SMTP_PASSWORD:-mail_password}

# This Dockerfile requires an exploded JasperReports Server WAR file installer file
# EXPLODED_INSTALLER_DIRECTORY (default jasperreports-server-bin/) directory below the Dockerfile.

RUN mkdir -p /usr/src/jasperreports-server

# get the WAR and license
COPY ${EXPLODED_INSTALLER_DIRECTORY}/jasperserver $CATALINA_HOME/webapps/jasperserver/
COPY ${EXPLODED_INSTALLER_DIRECTORY}/TIB* ${JRS_HOME}

# js-ant script, Ant XMLs and support in bin
COPY ${EXPLODED_INSTALLER_DIRECTORY}/buildomatic ${JRS_HOME}/buildomatic/

# Ant
COPY ${EXPLODED_INSTALLER_DIRECTORY}/apache-ant ${JRS_HOME}/apache-ant/

COPY scripts /

# Copy Fonts Extension lib
COPY assets/libs $CATALINA_HOME/webapps/jasperserver/WEB-INF/lib/

RUN chmod +x /*.sh && \
    /installPackagesForJasperserver-ce.sh > /dev/null && \
    echo "finished installing packages" && \
    rm -rf $CATALINA_HOME/webapps/ROOT && \
#    mkdir $CATALINA_HOME/webapps/ROOT && \
#    mv /index.jsp $CATALINA_HOME/webapps/ROOT/ && \
    rm -rf $CATALINA_HOME/webapps/docs && \
    rm -rf $CATALINA_HOME/webapps/examples && \
    rm -rf $CATALINA_HOME/webapps/host-manager && \
    rm -rf $CATALINA_HOME/webapps/manager && \
#    cp -R /buildomatic ${JRS_HOME}/buildomatic && \
    rm /installPackagesForJasperserver*.sh && rm -rf /buildomatic && \
    chmod +x ${JRS_HOME}/buildomatic/js-* && \
    chmod +x ${JRS_HOME}/apache-ant/bin/* && \
    wget "https://jdbc.postgresql.org/download/postgresql-${POSTGRES_JDBC_DRIVER_VERSION}.jar" \
        -P ${JRS_HOME}/buildomatic/conf_source/db/postgresql/jdbc --no-verbose --no-check-certificate && \
    keytool -genkey -alias self_signed -dname "CN=${DN_HOSTNAME}" \
        -storetype PKCS12 \
        -storepass "${KS_PASSWORD}" \
        -keypass "${KS_PASSWORD}" \
        -keystore /root/.keystore.p12 && \
    keytool -list -keystore /root/.keystore.p12 -storepass "${KS_PASSWORD}" -storetype PKCS12 && \
    xmlstarlet ed --inplace --subnode "/Server/Service" --type elem \
        -n Connector -v "" --var connector-ssl '$prev' \
    --insert '$connector-ssl' --type attr -n port -v "${HTTPS_PORT}" \
    --insert '$connector-ssl' --type attr -n protocol -v \
        "org.apache.coyote.http11.Http11NioProtocol" \
    --insert '$connector-ssl' --type attr -n maxThreads -v "150" \
    --insert '$connector-ssl' --type attr -n SSLEnabled -v "true" \
    --insert '$connector-ssl' --type attr -n scheme -v "https" \
    --insert '$connector-ssl' --type attr -n secure -v "true" \
    --insert '$connector-ssl' --type attr -n clientAuth -v "false" \
    --insert '$connector-ssl' --type attr -n sslProtocol -v "TLS" \
    --insert '$connector-ssl' --type attr -n keystorePass \
        -v "${KS_PASSWORD}" \
    --insert '$connector-ssl' --type attr -n keystoreFile \
        -v "/root/.keystore.p12" \
    ${CATALINA_HOME}/conf/server.xml

EXPOSE ${HTTP_PORT} ${HTTPS_PORT}

ENTRYPOINT ["/entrypoint-ce.sh"]

CMD ["run"]
