ARG LOGSTASH_VERSION=9.4.4
FROM docker.elastic.co/logstash/logstash:${LOGSTASH_VERSION}

ARG LOGSTASH_VERSION
ARG LOGSTASH_OUTPUT_HTTP_VERSION
LABEL org.opencontainers.image.base.version="$LOGSTASH_VERSION"

RUN if [ -n "$LOGSTASH_OUTPUT_HTTP_VERSION" ]; then \
        bin/logstash-plugin install --version "$LOGSTASH_OUTPUT_HTTP_VERSION" logstash-output-http; \
    else \
        bin/logstash-plugin install logstash-output-http; \
    fi && \
    bin/logstash-plugin list --verbose logstash-output-http
