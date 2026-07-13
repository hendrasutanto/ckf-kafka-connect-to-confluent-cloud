# Kafka Connect to Confluent Cloud

In this example, you'll set up a self-managed Kafka Connect cluster connected to Confluent Cloud, and install and manage the MySQL source connector plugin through the declarative `Connector` CRD.
Note: Here you'll only deploy Kafka Connect

## Set up Pre-requisites

Set the tutorial directory for this tutorial under the directory you downloaded the tutorial files:

```
export TUTORIAL_HOME=<Tutorial directory>/ccloud/connect
```

Create namespace

```
kubectl create ns confluent
```

## Deploy Confluent for Kubernetes

This workflow scenario assumes you are using the namespace `confluent`.

Set up the Helm Chart:

```
helm repo add confluentinc https://packages.confluent.io/helm
```

Install Confluent For Kubernetes using Helm:

```
helm upgrade --install operator confluentinc/confluent-for-kubernetes -n confluent
```
 
Check that the Confluent For Kubernetes pod comes up and is running:

```
kubectl get pods -n confluent
```

## Create Kubernetes Secrets for Confluent Cloud API Key and Confluent Cloud Schema Registry API Key

```
kubectl create secret generic ccloud-credentials --from-file=plain.txt=ccloud-credentials.txt
```

```
kubectl create secret generic ccloud-sr-credentials --from-file=basic.txt=ccloud-sr-credentials.txt
```

## Deploy self-managed Kafka Connect connecting to Confluent Cloud

```
kubectl apply -f $TUTORIAL_HOME/kafka-connect.yaml
```

## Port Forward REST endpoint for Kafka Connect to submit Connector config

```
kubectl port-forward connect-0 8083
```

## Create Connector

Create MySQL source connector

Note: Because Kafka lives in Confluent Cloud (not a local broker), the connector's internal
schema history topic also needs its own SASL_SSL credentials for its producer and consumer.
Replace `<ccloud-api-key>` and `<ccloud-api-secret>` with the same Confluent Cloud Kafka API
key/secret used in `ccloud-credentials.txt`.

```
curl -X PUT \
-H "Content-Type: application/json" \
--data '{
	"connector.class":"io.debezium.connector.mysql.MySqlConnector",
	"tasks.max":"1",
	"database.hostname":"<mysql_host>",
	"database.port":"3306",
	"database.user":"<mysql_user>",
	"database.password":"<mysql_password>",
	"database.server.id":"<unique_numeric_id>",
	"database.include.list":"<database_to_import>",
	"topic.prefix":"ccloud-",
	"schema.history.internal.kafka.topic":"schema-changes.mysql",
	"schema.history.internal.kafka.bootstrap.servers":"<ccloud_bootstrap_endpoint>",
	"schema.history.internal.producer.security.protocol":"SASL_SSL",
	"schema.history.internal.producer.sasl.mechanism":"PLAIN",
	"schema.history.internal.producer.sasl.jaas.config":"org.apache.kafka.common.security.plain.PlainLoginModule required username=\"<ccloud-api-key>\" password=\"<ccloud-api-secret>\";",
	"schema.history.internal.consumer.security.protocol":"SASL_SSL",
	"schema.history.internal.consumer.sasl.mechanism":"PLAIN",
	"schema.history.internal.consumer.sasl.jaas.config":"org.apache.kafka.common.security.plain.PlainLoginModule required username=\"<ccloud-api-key>\" password=\"<ccloud-api-secret>\";",
	"key.converter.basic.auth.credentials.source": "USER_INFO",
	"key.converter.schema.registry.basic.auth.user.info":"<ccloud-sr-api-key>:<ccloud-sr-api-secret>",
	"key.converter.schema.registry.url": "<ccloud-sr-endpoint>",
	"value.converter.basic.auth.credentials.source": "USER_INFO",
	"value.converter.schema.registry.basic.auth.user.info":"<ccloud-sr-api-key>:<ccloud-sr-api-secret>",
	"value.converter.schema.registry.url": "<ccloud-sr-endpoint>"
}' \
http://localhost:8083/connectors/mysql-source-ccloud/config | jq .
```

## Validation
Check if connector is running
```
curl -X GET http://localhost:8083/connectors/mysql-source-ccloud
```

## Tear down
```
kubectl delete -f $TUTORIAL_HOME/kafka-connect.yaml
```
