# Demo

This is the official bwNetFlow demo and should give you an introduction to your project and it's usage.

This demo is based on containerized applications and it's dependencies. For further information and configuration of the third party applications we use, please refer to their documentation too.  

## Structure

This demo is split into two parts.  
The first part is for **operators** to deploy the whole bwnetFlow Project. The second part is the **customer** specific deployment, which is meant to consume data from an existing deployment of the project.

## Getting Started

### Quickstart

1. optional: configure authentication by following the instructions in `kafka_auth.env`
2. `docker-compose up -d`
3. access http://localhost:3000 login as user `demo` with password `demo` and look at dashboards

If you're curious about some internals, Prometheus' web interface is available
at http://localhost:9090 and the exported data scraped from
http://localhost:8080/metrics and http://localhost:8080/flowdata respectively.

## Deployment for Operators

This section gives a more detailed description for the **operators** deployment. Third party application doesn't have any sophisticated configuration. Also be aware that we didn't provide any solutions for persisting your data.  

In the following some comments about the used applications in the order they occur in the `docker-compose.yml` file.  

**flowexport**:  
This is for turning traffic gathered on the given interface into Netflow.
If your configuring Netflow on your router, you won't need this section.
This tool can be also useful for testing purposes or any demo deployment.

**goflow**:  
We need goflow as ingress for all gathered netflow data. These flowdata are written into the `flows` topic of the kafka instance.

**kafka**:  
This is a minimal single node kafka deployment with zookeeper. The `initializer` is needed to create the topic for the tools which can't create topics.

**enricher**:  
The enricher consumes the raw flows of the `flows` topic from kafka and enriches them with additional data.
These enriched flows are then written back to the `flows-enriched` topic.
As you can see, we usually use TLS and SASL in producton, but we disable it for these dev setups.
For further information check out the [processor_enricher](https://github.com/bwNetFlow/processor_enricher) repository.

**splitter**:  
This tool was not originally in any of our dev setups.
The CIDS whitelist tells the splitter to create flows-customer-100,
flows-customer-101, ... topics. If you wish to split on another field
than Cid (because there is none...), you'd have to adapt line 110 and 111
of the splitters main.go. Note that the field should be an integer for now.
For a dive in check out the repo: [processor_splitter](https://github.com/bwNetFlow/processor_splitter) 

A docker-compose snipped for this tool will look like this:  

```yaml
version: '3'
services:
  splitter:
    image: ghcr.io/bwnetflow/processor_splitter:latest
    stdin_open: true
    tty: true
    restart: unless-stopped
    environment:
      KAFKA_BROKERS: kafka.local:9092
      KAFKA_IN_TOPIC: flows-enriched
      KAFKA_OUT_TOPICPREFIX: flows-customer
      KAFKA_CONSUMER_GROUP: test-consumer-group
      DISABLE_AUTH: 'true'
      DISABLE_TLS: 'true'
      AUTH_ANON: 'false'
      CIDS: "100,101,102"
```

**reducer**:  
This tool was not originally in any of our dev setups. Please check out the repository for additional information ([processor_reducer](https://github.com/bwNetFlow/processor_reducer)).

A docker-compose snipped for this tool will look like this:  

```yaml
version: '3'
services:
  reducer:
    image: ghcr.io/bwnetflow/processor_reducer:latest
    stdin_open: true
    tty: true
    restart: unless-stopped
    environment:
      KAFKA_BROKERS: kafka.local:9092
      KAFKA_IN_TOPIC: flows-enriched
      KAFKA_OUT_TOPIC: flows-anon
      KAFKA_CONSUMER_GROUP: reducer-prod
      DISABLE_AUTH: 'true'
      DISABLE_TLS: 'true'
      AUTH_ANON: 'false'
```

**consumer**:
This can be the same consumer as in the demo ([consumer_prometheus](https://github.com/bwNetFlow/consumer_prometheus)), or anything geared towards your use case.
In the demo we have on github, this acts as a prometheus exporter and is scraped by below prometheus.
The demo also uses the enriched topic directly and does not bother with splitting it beforehand.
It really depends on what your intentions are at this point.
We have our users implement Consumers themselves, in a language of their choice.
In addition, we provide some limited dashboards, they're however
generated from the enriched topic directly.

**prometheus**:  
This is part of the frontend, unessential for the setup. Prometheus will scrape data from the given consumer.
In the **operators** setup, prometheus is also configured to get data directly from goflow for debugging purpose. 

**grafana**:  
Using the official container too. In addition we provide some dashboards and the `goflow-internals` dashboard.

## Deployment for Customers

The deployment for customers only needs three components from the above mentioned.
These components are **consumer, prometheus and grafana**.
It's meant to consume flows from a given kafka topic and provides them to a dedicated prometheus instance.
Some default grafana dashboards and a minimal prometheus configuration are also provided.  

The demo deployment is configured without TLS and SASL authentication.
To set up a connection to a remote kafka cluster with TLS and auth this could look like the following compose snippet.

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