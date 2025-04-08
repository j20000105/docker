FROM docker.elastic.co/logstash/logstash:8.17.4
RUN echo 'api.http.host: "0.0.0.0"' > /usr/share/logstash/config/logstash.yml
RUN bin/logstash-plugin install logstash-output-http
