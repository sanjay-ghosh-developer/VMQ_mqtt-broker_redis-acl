FROM ubuntu

RUN apt-get update && \
    apt-get -y install bash procps openssl iproute2 curl jq libsnappy-dev net-tools nano && \
    rm -rf /var/lib/apt/lists/* && \
    addgroup --gid 10000 vernemq && \
    adduser --uid 10000 --system --ingroup vernemq --home /vernemq --disabled-password vernemq
RUN echo "session required        pam_limits.so"  >>  /etc/pam.d/common-session && \
    echo "*               soft     nofile          65536\n*               hard     nofile          65536"  >>  /etc/security/limits.conf
RUN sysctl -w net.ipv4.tcp_rmem="4096 16384 32768" && \
    sysctl -w net.ipv4.tcp_wmem="4096 16384 32768"

# Nope, these values are not recommendations!
# You really need to decide yourself.
WORKDIR /vernemq

# Defaults
ENV DOCKER_VERNEMQ_KUBERNETES_LABEL_SELECTOR="app=vernemq" \
    DOCKER_VERNEMQ_LOG__CONSOLE=console \
    PATH="/vernemq/bin:$PATH"

COPY --chown=10000:10000 vernemq_binary.tar.gz /vernemq/vernemq_binary.tar.gz
RUN tar -xvf /vernemq/vernemq_binary.tar.gz && \
    rm /vernemq/vernemq_binary.tar.gz
    
COPY --chown=10000:10000 vernemq.sh /usr/sbin/start_vernemq
COPY --chown=10000:10000 vm.args /vernemq/etc/vm.args

RUN chown -R 10000:10000 /vernemq && \
    ln -s /vernemq/etc /etc/vernemq && \
    ln -s /vernemq/data /var/lib/vernemq && \
    ln -s /vernemq/log /var/log/vernemq

# Ports
# 1883  MQTT
# 8883  MQTT/SSL
# 8080  MQTT WebSockets
# 44053 VerneMQ Message Distribution
# 4369  EPMD - Erlang Port Mapper Daemon
# 8888  Prometheus Metrics
# 9100 9101 9102 9103 9104 9105 9106 9107 9108 9109  Specific Distributed Erlang Port Range

EXPOSE 1883 8883 8080 44053 4369 8888 \
       9100 9101 9102 9103 9104 9105 9106 9107 9108 9109


VOLUME ["/vernemq/log", "/vernemq/data", "/vernemq/etc"]

HEALTHCHECK CMD vernemq ping | grep -q pong

USER vernemq

CMD ["start_vernemq"]
