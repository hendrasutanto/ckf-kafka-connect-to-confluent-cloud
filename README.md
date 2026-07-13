# CFK Managed Connect Cluster with Confluent Hub Connector, Connected to Confluent Cloud

This scenario deploys a self-managed Kafka Connect cluster via Confluent for
Kubernetes (CFK), running the Debezium MySQL CDC Source connector, and
connects it to a Kafka cluster and Schema Registry in Confluent Cloud.

## Prerequisites

- A Kubernetes cluster with CFK installed (this example was validated
  against CFK 3.3.0 / Confluent Platform 8.2.0)
- A Confluent Cloud environment with a Kafka cluster and Schema Registry
  enabled
- A Confluent Cloud API key/secret for Kafka, and a separate API
  key/secret for Schema Registry
- A reachable MySQL instance with row-based binary logging enabled
  (`log_bin=ON`, `binlog_format=ROW`)

## Setup

1. Set your tutorial directory:

   ```
   export TUTORIAL_HOME=<path-to-this-directory>
   ```

2. Fill in `ccloud-credentials.txt` and `ccloud-sr-credentials.txt` with
   your Confluent Cloud API key/secret pairs.

3. Create the Kubernetes secrets:

   ```
   kubectl create secret generic ccloud-credentials \
     --from-file=plain.txt=$TUTORIAL_HOME/ccloud-credentials.txt \
     --namespace confluent

   kubectl create secret generic ccloud-sr-credentials \
     --from-file=basic.txt=$TUTORIAL_HOME/ccloud-sr-credentials.txt \
     --namespace confluent
   ```

4. Edit `kafka-connect.yaml` and replace `<replace-with-ccloud-bootstrap>`
   and `<replace-with-ccloud-sr-url>` with your Confluent Cloud endpoints.

5. Apply the Connect CR:

   ```
   kubectl apply -f $TUTORIAL_HOME/kafka-connect.yaml --namespace confluent
   ```

6. Confirm the Connect pod comes up and the connector plugin installed
   successfully:

   ```
   kubectl get pods --namespace confluent
   kubectl logs -f connect-0 -c config-init-container --namespace confluent
   ```

## Next steps

Once Connect is running, deploy a `Connector` custom resource for the
Debezium MySQL source connector to start streaming change events into
your Confluent Cloud cluster.
