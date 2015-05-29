# wlp_sti
FROM pnavascues/wlp

ENV WLP_SERVER_HOME /opt/ibm/wlp/usr/servers/defaultServer
ENV STI_SCRIPTS_URL image://.sti/bin/

ADD .sti .sti
RUN chmod +x /.sti/bin/*

# No incremental builds, so we don't need this
# RUN yum install -y tar gunzip &&  yum clean all -y

# USER default

# TODO (optional): Copy the builder files into /opt/openshift
# COPY ./<builder_folder>/ /opt/openshift/

# TODO: Set the default port for applications built using this image

EXPOSE 9080
EXPOSE 9443
