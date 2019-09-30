This is the official bwNetFlow demo and a beta for the official container
provided to BelWü participants.

1. build or pull the docker image:

```
# pull:
# TODO: docker pull ????

# build:
cd consumer
make # needs a working golang env, will create bwnetflow:latest
```

2. configure authentication by following the instructions in `kafka_auth.env`
3. `docker-compose up`
4. access http://localhost:3000 login as user `demo` with password `demo` and look at dashboards

If you're curious about some internals, Prometheus' web interface is available
at http://localhost:9090 and the exported data scraped from
http://localhost:8080/metrics and http://localhost:8080/flowdata respectively.
