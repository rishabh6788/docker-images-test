FROM amazonlinux:2 AS linux_stage_0

ARG UID=1000
ARG GID=1000
ARG TEMP_DIR=/tmp/opensearch-dashboards
ARG OPENSEARCH_DASHBOARDS_HOME=/usr/share/opensearch-dashboards
ARG OSD_VERSION=2.0.1

# Update packages
# Install the tools we need: tar and gzip to unpack the OpenSearch tarball, and shadow-utils to give us `groupadd` and `useradd`.
# Install which to allow running of securityadmin.sh
RUN yum update -y && yum install -y tar gzip shadow-utils which && yum clean all

# Create an opensearch-dashboards user, group, and directory
RUN groupadd -g $GID opensearch-dashboards && \
    adduser -u $UID -g $GID -d $OPENSEARCH_DASHBOARDS_HOME opensearch-dashboards && \
    mkdir $TEMP_DIR

RUN set -eux ; \
    cur_arch="" ; \
    case "$(arch)" in \
        aarch64) cur_arch='arm64' ;; \
        x86_64)  cur_arch='x64' ;; \
        *) echo >&2 ; echo >&2 "Unsupported architecture $(arch)" ; echo >&2 ; exit 1 ;; \
    esac ; \
    curl --retry 10 -S -L --output $TEMP_DIR/opensearch-dashboard.tar.gz https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/$OSD_VERSION/opensearch-dashboards-$OSD_VERSION-linux-$cur_arch.tar.gz; \
    curl --output $TEMP_DIR/opensearch.pgp https://artifacts.opensearch.org/publickeys/opensearch.pgp; \
    gpg --import $TEMP_DIR/opensearch.pgp; \
    curl --output $TEMP_DIR/opensearch-dashboard.tar.gz.sig https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/$OSD_VERSION/opensearch-dashboards-$OSD_VERSION-linux-$cur_arch.tar.gz.sig; \
    gpg --verify $TEMP_DIR/opensearch-dashboard.tar.gz.sig $TEMP_DIR/opensearch-dashboard.tar.gz; \
    tar --warning=no-timestamp -zxf $TEMP_DIR/opensearch-dashboard.tar.gz -C $OPENSEARCH_DASHBOARDS_HOME --strip-components=1;

COPY --chown=opensearch-dashboards:opensearch-dashboards config/* $OPENSEARCH_DASHBOARDS_HOME/config/
COPY --chown=opensearch-dashboards:opensearch-dashboards bin/* $OPENSEARCH_DASHBOARDS_HOME/

########################### Stage 1 ########################
# Copy working directory to the actual release docker images
FROM amazonlinux:2

ARG UID=1000
ARG GID=1000
ARG OPENSEARCH_DASHBOARDS_HOME=/usr/share/opensearch-dashboards
ARG OSD_VERSION=2.0.1

# Update packages
# Install the tools we need: tar and gzip to unpack the OpenSearch tarball, and shadow-utils to give us `groupadd` and `useradd`.
# Install which to allow running of securityadmin.sh
RUN yum update -y && yum install -y tar gzip shadow-utils which \
    libnss3.so xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-utils \
    xorg-x11-fonts-cyrillic xorg-x11-fonts-Type1 xorg-x11-fonts-misc fontconfig freetype && \
    yum clean all

RUN groupadd -g $GID opensearch-dashboards && \
    adduser -u $UID -g $GID -d $OPENSEARCH_DASHBOARDS_HOME opensearch-dashboards

COPY --from=linux_stage_0 --chown=$UID:$GID $OPENSEARCH_DASHBOARDS_HOME $OPENSEARCH_DASHBOARDS_HOME

# Setup OpenSearch-dashboards
WORKDIR $OPENSEARCH_DASHBOARDS_HOME

# Set PATH
ENV PATH=$PATH:$OPENSEARCH_DASHBOARDS_HOME/bin

# Change user
USER $UID

# Expose port
EXPOSE 5601

LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.name="opensearch-dashboards" \
  org.label-schema.version="$OSD_VERSION" \
  org.label-schema.url="https://opensearch.org" \
  org.label-schema.vcs-url="https://github.com/opensearch-project/OpenSearch-Dashboards" \
  org.label-schema.license="Apache-2.0" \
  org.label-schema.vendor="Amazon"

ENTRYPOINT ["./opensearch-dashboards-docker-entrypoint.sh"]
CMD ["opensearch-dashboards"]
