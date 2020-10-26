# Getting Started

## Quickstart

1. optional: configure authentication by following the instructions in `kafka_auth.env`
2. `docker-compose up -d`
3. access http://localhost:3000 login as user `demo` with password `demo` and look at dashboards

If you're curious about some internals, Prometheus' web interface is available
at http://localhost:9090 and the exported data scraped from
http://localhost:8080/metrics and http://localhost:8080/flowdata respectively.
 
## Deployment for Customers

The deployment for customers only needs three components mentioned in the operators documentation.
These components are **consumer, prometheus and grafana**.
It's meant to consume flows from a given kafka topic and provides them to a dedicated prometheus instance.
Some default grafana dashboards and a minimal prometheus configuration are also provided.  

The demo deployment is configured without TLS and SASL authentication.
To set up a connection to a remote kafka cluster with TLS and auth this could look like the following compose snippet. For further information regarding the consumer please check out [consumer_prometheus](https://github.com/bwNetFlow/consumer_prometheus).

```yaml
  consumer:
    image: ghcr.io/bwnetflow/consumer_prometheus:latest
    stdin_open: true
    tty: true
    restart: unless-stopped
    environment:
      KAFKA_BROKERS: kafka.remote:9092
      KAFKA_TOPIC: flow-topic-to-read
      KAFKA_USER: username
      KAFKA_PASS: ultraSecretPassword
      KAFKA_CONSUMER_GROUP: your-consumer-group
```

## Notes

In order to configure the authentication variables in your compose file you can also use an `env_file` instead of the dependent variables.

```yaml
    env_file:
      - .kafka_auth.env # TODO: follow the instructions in kafka_auth.env
```
