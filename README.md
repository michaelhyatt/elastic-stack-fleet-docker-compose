# Running ES, Kibana and Fleet server in docker-compose

## Intro
This setup runs ES, Kibana and Fleet server in docker-compose with auto-generated self-signed certificates and full security turned on that supports Kibana alerts. After starting docker-compose, it will gradually start ES with Kibana, then will bring up Fleet server and register it with Kibana. In this state, it is ready to start external agent enrollment with `--insecure` flag due to the self-signed certificates it is using. This setup is also suitable for full on-prem stack PoCs where a single server install with ES, Kibana and Fleet server is enough, but will have all the enterprise features enabled (including Kibana Alerting) with a trial license automatically turned on.

## Setup
The current setup is specifying ES and Fleet server endpoints as `localhost`. It is possible to change it to the real hostname of the server in the `.env` file.

```bash
# Externally accessible URLs of ES and Fleet servers
FLEET_URL=https://localhost:8220
ES_URL=https://localhost:9200
```

## Running it
To start the whole stack:
```bash
docker-compose up -d
```
Output:
```bash
Creating network "docker-compose_default" with the default driver
Creating volume "docker-compose_certs" with local driver
Creating docker-compose_setup_1 ... done
Creating docker-compose_es01_1  ... done
Creating docker-compose_kib01_1 ... done
Creating fleet                  ... done
```
Once the stack is up, log into Kibana (https://localhost:5601 by default) with the credentials specified in the `.env` file (elastic:changeme is the default). Under `Add Agent` flyout in Fleet UI, use the provided commands to install and enroll the agent, but add the `--insecure` flag to the command. Change the Fleet server url, if needed.

```bash
$ sudo ./elastic-agent install --url=https://localhost:8220 --enrollment-token=OXA4...0VnakRVTVdTZw== --insecure
```
Output:
```bash
Elastic Agent will be installed at /Library/Elastic/Agent and will run as a service. Do you want to continue? [Y/n]:
{"log.level":"warn","@timestamp":"2022-08-13T12:29:16.034+1000","log.logger":"tls","log.origin":{"file.name":"tlscommon/tls_config.go","file.line":104},"message":"SSL/TLS verifications disabled.","ecs.version":"1.6.0"}
{"log.level":"info","@timestamp":"2022-08-13T12:29:16.550+1000","log.origin":{"file.name":"cmd/enroll_cmd.go","file.line":471},"message":"Starting enrollment to URL: https://localhost:8220/","ecs.version":"1.6.0"}
{"log.level":"warn","@timestamp":"2022-08-13T12:29:16.664+1000","log.logger":"tls","log.origin":{"file.name":"tlscommon/tls_config.go","file.line":104},"message":"SSL/TLS verifications disabled.","ecs.version":"1.6.0"}
{"log.level":"info","@timestamp":"2022-08-13T12:29:17.500+1000","log.origin":{"file.name":"cmd/enroll_cmd.go","file.line":273},"message":"Successfully triggered restart on running Elastic Agent.","ecs.version":"1.6.0"}
Successfully enrolled the Elastic Agent.
Elastic Agent has been successfully installed.
```