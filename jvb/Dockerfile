ARG JITSI_REPO=jitsi
FROM ${JITSI_REPO}/base-java

ADD https://download.jitsi.org/stable/jitsi-videobridge2_2.1-197-g38256192-1_all.deb /jvb.deb
ADD https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz /

RUN tar xvf /node_exporter-1.0.1.linux-amd64.tar.gz && \
    mv /node_exporter-1.0.1.linux-amd64/node_exporter /node_exporter && \
    rm -rf /node_exporter-1.0.1.linux-amd64* && \
    chown 1001:0 /node_exporter && \
    chmod g=u /node_exporter

RUN \
	apt-dpkg-wrap apt-get update && \
	apt-dpkg-wrap apt install -y /jvb.deb && \
	apt-cleanup

COPY rootfs/ /

RUN /fix-permissions.sh

VOLUME /config

USER 1001
