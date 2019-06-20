FROM docker.elastic.co/logstash/logstash:6.4.0

ADD pipelines.yml /usr/share/logstash/config/
ADD config/ /etc/logstash/conf.d/
ADD patterns /etc/logstash/conf.d/patterns
ADD GeoLite2-City.mmdb /etc/logstash/

#ADD logstash.crt /etc/pki/tls/certs/
#ADD logstash.key /etc/pki/tls/private/


