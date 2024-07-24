FROM jboss/wildfly:20.0.1.Final
LABEL maintainer="jorgelojam@gmail.com"

ENV WILDFLY_USER jloja
ENV WILDFLY_PASS JlojaPassword

ENV DS_NAME KitchensinkAngularJSQuickstartDS
ENV DS_USER consultas
ENV DS_PASS QueryConSql.pwd
ENV DS_URI jdbc:postgresql://srvdb/consultas

ENV JBOSS_CLI $JBOSS_HOME/bin/jboss-cli.sh
ENV DEPLOYMENT_DIR $JBOSS_HOME/standalone/deployments/

RUN echo "Adding WildFly administrator"
RUN $JBOSS_HOME/bin/add-user.sh -u $WILDFLY_USER -p $WILDFLY_PASS --silent

# Configure Wildfly server
RUN echo "Starting WildFly server" && \
      bash -c '$JBOSS_HOME/bin/standalone.sh -c standalone.xml &' && \
      bash -c 'until `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null | grep -q running`; do echo `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null`; sleep 1; done' && \
      curl --location --output /tmp/postgresql-42.2.16.jar --url https://jdbc.postgresql.org/download/postgresql-42.2.16.jar && \
      $JBOSS_CLI --connect --command="module add --name=org.postgres --resources=/tmp/postgresql-42.2.16.jar --dependencies=javax.api,javax.transaction.api" && \
      $JBOSS_CLI --connect --command="/subsystem=datasources/jdbc-driver=postgres:add(driver-name="postgres",driver-module-name="org.postgres",driver-class-name=org.postgresql.Driver)" && \
      $JBOSS_CLI --connect --command="data-source add \
        --name=${DS_NAME} \
        --jndi-name=java:jboss/datasources/${DS_NAME} \
        --user-name=${DS_USER} \
        --password=${DS_PASS} \
        --driver-name=postgres \
        --connection-url=${DS_URI} \
        --min-pool-size=10 \
        --max-pool-size=20 \
        --blocking-timeout-wait-millis=5000 \
        --statistics-enabled=true \
        --enabled=true" && \
      $JBOSS_CLI --connect --command=":shutdown" && \
      rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history/ $JBOSS_HOME/standalone/log/* && \
      rm -f /tmp/*.jar

COPY target/kitchensink-angularjs.war /opt/jboss/wildfly/standalone/deployments/
EXPOSE 8080
EXPOSE 9990