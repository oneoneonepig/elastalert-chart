# Elastalert Helm Chart

[elastalert](https://github.com/Yelp/elastalert) is a simple framework for alerting on anomalies, spikes, or other patterns of interest from data in Elasticsearch.

## Pre-configs and Versions

* Kubernetes - 1.14.6
* ELK - 7.3.2
* ElastAlert - 0.2.1
* Helm - 2.14.3
* Elasticsearch Endpoint -
  * host - elasticsearch-master:9200
  * TLS - false
* SMTP settings -
  * smtp_host - smtp.sendgrid.net
  * smtp_port - 587
  * smtp_auth_file - /opt/config-smtp/smtp-auth.yaml
  * from_addr - elastalert@miniasp.com
  * notify_email - peihua@miniasp.com
* SMTP credential secret object - elastalert-smtp-auth
* SMTP credential secret file name - smtp-auth.yaml
* Run interval - 1 minute
* Buffer time - 15 minutes

## (If needed) Install ELK Stack

```console
helm install --name elasticsearch --namespace elk elastic/elasticsearch
helm install --name kibana --namespace elk elastic/kibana \
--set service.type=LoadBalancer
helm install --name filebeat --namespace elk elastic/filebeat
```

## Installing the Chart;

```console
# Add user and password entry for smtp credentials

vim smtp-auth.yaml
kubectl create secret generic -n elk elastalert-smtp-auth --from-file=./smtp_auth.yaml

# Install chart

helm install --name elastalert --namespace elk .
```

## Uninstalling the Chart

```console
helm delete --purge elastalert
kubectl delete secret -n elk elastalert-smtp-auth
```

## Some Scripts

```console
# Create job that echo bad every 5 seconds

kubectl create job --image=alpine echo-bad -- /bin/sh -c "while true; do echo bad; sleep 5; done"

# Create job that echo worse every 5 seconds

kubectl create job --image=alpine echo-worse -- /bin/sh -c "while true; do echo worse; sleep 5; done"

# Testing rules

kubectl exec -n elk $(kubectl get pod -n elk -l app=elastalert -o name) -- \
elastalert-test-rule --config=../config/elastalert_config.yaml ../rules/bad_output.yaml

# Testing rules, 1 hour ago

kubectl exec -n elk -it $(kubectl get pod -n elk -l app=elastalert -o name) -- \
elastalert-test-rule --config=../config/elastalert_config.yaml ../rules/bad_output.yaml --start $(date -u --date="1 hour ago" +%Y-%m-%dT%H:%M:%S)

# Listing indices

kubectl exec -n elk elasticsearch-master-0 -- curl -s localhost:9200/_cat/indices?v
```

## Configuration

|       Parameter          |                    Description                    |             Default             |
| ------------------------ | ------------------------------------------------- | ------------------------------- |
| `image.repository`       | docker image                                      | jertel/elastalert-docker        |
| `image.tag`              | docker image tag                                  | 0.2.1                           |
| `image.pullPolicy`       | image pull policy                                 | IfNotPresent                    |
| `command`                | command override for container                    | `NULL`                          |
| `args`                   | args override for container                       | `NULL`                          |
| `replicaCount`           | number of replicas to run                         | 1                               |
| `elasticsearch.host`     | elasticsearch endpoint to use                     | elasticsearch                   |
| `elasticsearch.port`     | elasticsearch port to use                         | 80                              |
| `elasticsearch.useSsl`   | whether or not to connect to es_host using SSL    | False                           |
| `elasticsearch.username` | Username for ES with basic auth                   | `NULL`                          |
| `elasticsearch.password` | Password for ES with basic auth                   | `NULL`                          |
| `elasticsearch.verifyCerts` | whether or not to verify TLS certificates    | True                            |
| `elasticsearch.clientCert` | path to a PEM certificate to use as the client certificate | /certs/client.pem  |
| `elasticsearch.clientKey` | path to a private key file to use as the client key | /certs/client-key.pem      |
| `elasticsearch.caCerts` | path to a CA cert bundle to use to verify SSL connections | /certs/ca.pem          |
| `elasticsearch.certsVolumes` | certs volumes, required to mount ssl certificates when elasticsearch has tls enabled | `NULL` |
| `elasticsearch.certsVolumeMounts` | mount certs volumes, required to mount ssl certificates when elasticsearch has tls enabled | `NULL` |
| `extraConfigOptions` | Additional options to propagate to all rules, cannot be `alert`, `type`, `name` or `index` | `{}` |
| `extraVolumes`           | Additional volume definitions                     | []                              |
| `extraVolumeMounts`      | Additional volumeMount definitions                | []                              |
| `resources`              | Container resource requests and limits            | {}                              |
| `rules`                  | Rule and alert configuration for Elastalert       | {} example shown in values.yaml |
| `runIntervalMins`        | Default interval between alert checks, in minutes | 1                               |
| `realertIntervalMins`    | Time between alarms for same rule, in minutes     | `NULL`                          |
| `alertRetryLimitMins`    | Time to retry failed alert deliveries, in minutes | 2880 (2 days)                   |
| `bufferTimeMins`         | Default rule buffer time, in minutes              | 15                              |
| `writebackIndex`         | Name or prefix of elastalert index(es)            | elastalert_status               |
| `nodeSelector`           | Node selector for deployment                      | {}                              |
| `tolerations`            | Tolerations for deployment                        | []                              |
