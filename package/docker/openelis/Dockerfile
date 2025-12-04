FROM amazoncorretto:8

ENV SERVER_PORT=8052
ENV BASE_DIR=/var/run/bahmni-lab
ENV CONTEXT_PATH=/openelis
ENV WAR_DIRECTORY=${BASE_DIR}/bahmni-lab
ENV SERVER_OPTS="-Xms512m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m"
ENV DEBUG_OPTS="-agentlib:jdwp=transport=dt_socket,address=8001,server=y,suspend=n"
ENV OPENELIS_DB_SERVER='openelisdb'

# -------------------------------------------------------
# Dependencies
# -------------------------------------------------------
RUN mkdir -p ${WAR_DIRECTORY} \
    && mkdir -p /opt/bahmni-lab/migrations/db_backup/ \
    && mkdir -p /opt/bahmni-lab/migrations/liquibase/ \
    && mkdir -p /opt/bahmni-lab/migrations/scripts/ \
    && mkdir -p /home/bahmni/uploaded_results/ \
    && mkdir -p /home/bahmni/uploaded-files/elis/

RUN yum install -y unzip nc postgresql gettext

# -------------------------------------------------------
# Fetch utilities and WAR
# -------------------------------------------------------
ADD https://raw.githubusercontent.com/eficode/wait-for/v2.2.3/wait-for wait-for.sh
ADD https://repo.mybahmni.org/packages/build/bahmni-embedded-tomcat-8.0.42.jar \
    /opt/bahmni-lab/lib/bahmni-lab.jar

# Copy the built WAR file
COPY openelis/dist/openelis.war /etc/bahmni-lab/openelis.war

# -------------------------------------------------------
# Extract WAR
# -------------------------------------------------------
RUN cd ${WAR_DIRECTORY} && jar xvf /etc/bahmni-lab/openelis.war

# -------------------------------------------------------
# Templates + Scripts
# -------------------------------------------------------
COPY package/docker/openelis/templates/hibernate.cfg.xml.template /etc/bahmni-lab/
COPY package/docker/openelis/templates/atomfeed.properties.template /etc/bahmni-lab/
COPY package/resources/log4j2.xml ${WAR_DIRECTORY}/WEB-INF/classes/
COPY package/docker/openelis/update_openmrs_host_port.sh update_openmrs_host_port.sh
RUN chmod +x update_openmrs_host_port.sh
RUN chmod +x wait-for.sh

# -------------------------------------------------------
# OpenElis.zip (migrations)
# -------------------------------------------------------
COPY OpenElis.zip /tmp/artifacts/OpenElis.zip

RUN unzip -d /tmp/artifacts/ /tmp/artifacts/OpenElis.zip \
    && cp -r /tmp/artifacts/OpenElis/db_backup/. /opt/bahmni-lab/migrations/db_backup/ \
    && cp -r /tmp/artifacts/OpenElis/liquibase/. /opt/bahmni-lab/migrations/liquibase/ \
    && cp -r /tmp/artifacts/OpenElis/scripts/. /opt/bahmni-lab/migrations/scripts/ \
    && rm -rf /tmp/artifacts

COPY package/resources/migrateDb.sh /opt/bahmni-lab/migrations/scripts/
RUN chmod +x /opt/bahmni-lab/migrations/scripts/migrateDb.sh

COPY package/docker/openelis/run-implementation-openelis-implementation.sh \
     /opt/bahmni-lab/migrations/scripts/
RUN chmod +x /opt/bahmni-lab/migrations/scripts/run-implementation-openelis-implementation.sh

# -------------------------------------------------------
# Start Script
# -------------------------------------------------------
COPY package/docker/openelis/start.sh start.sh
RUN chmod +x start.sh

CMD ["./start.sh"]
