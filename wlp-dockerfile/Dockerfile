FROM rhel7.1

MAINTAINER Pablo Navascues  <pablo@redhat.com> (@pnavascues)

RUN yum -y install tar && yum -y clean all 

# Install JRE
ADD ibm-java-x86_64-sdk-7.1-2.10.bin /tmp/ibm-java.bin
RUN chmod +x /tmp/ibm-java.bin \
    && echo "INSTALLER_UI=silent" > /tmp/response.properties \
    && echo "USER_INSTALL_DIR=/opt/ibm/java" >> /tmp/response.properties \
    && mkdir /opt/ibm \
    && /tmp/ibm-java.bin -i silent -f /tmp/response.properties \
    && rm -f /tmp/response.properties \
    && rm -f /tmp/ibm-java.bin
ENV JAVA_HOME /opt/ibm/java
ENV PATH $JAVA_HOME/jre/bin:/opt/ibm/wlp/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LICENSE accept
COPY license-check /opt/ibm/docker/
COPY view-jre-license /opt/ibm/docker/licenses/

# Install WebSphere Liberty
COPY wlp-runtime-8.5.5.5.jar /tmp/wlp-developers-runtime.jar
RUN java -jar /tmp/wlp-developers-runtime.jar --acceptLicense /opt/ibm 
RUN rm -f /tmp/wlp-developers-runtime.jar 

COPY view-wlp-license /opt/ibm/docker/licenses/
COPY liberty-run /opt/ibm/wlp/bin/
ENV PATH /opt/ibm/wlp/bin:$PATH

# Configure WebSphere Liberty
RUN /opt/ibm/wlp/bin/server create \
    && rm -rf /opt/ibm/wlp/usr/servers/.classCache
COPY server.xml /opt/ibm/wlp/usr/servers/defaultServer/
RUN chmod +x /opt/ibm/docker/license-check \ 
    && chmod +x /opt/ibm/wlp/bin/liberty-run

EXPOSE 9080
EXPOSE 9443

ENTRYPOINT ["liberty-run"]
CMD ["/opt/ibm/wlp/bin/server", "run", "defaultServer"]

