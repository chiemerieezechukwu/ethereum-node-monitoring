# Ethereum Node Monitoring, Logging and Alerting

This stack runs an ethereum node (execution client and beacon node). It includes Prometheus, Grafana, Loki, and other tools for metrics collection, log aggregation, and alerting.

## Ethereum Clients

The setup includes two Ethereum clients: Geth and Lodestar. Both clients are needed for running an Ethereum node and interacting with the Ethereum blockchain.

- Geth (Go Ethereum): Geth is an Ethereum client that handles the execution layer of Ethereum. Listens to new transactions broadcasted in the network, executes them and holds the latest state and database of all current Ethereum data. CLI configuration flags are passed directly to `geth` as an array via the `command` key under the `execution_client` docker compose service. More CLI options can be found at [Geth Command-line Options](https://geth.ethereum.org/docs/fundamentals/command-line-options).

- Lodestar: Implements the proof-of-stake consensus algorithm, which enables the network to achieve agreement based on validated data from the execution client. Like the Geth service, CLI configuration flags are passed directly to `lodestar` as an array via the `command` key under the `beacon`docker compose service. More CLI options can be found at [Beacon CLI Command](https://chainsafe.github.io/lodestar/run/beacon-management/beacon-cli).

> [!NOTE]
> Both Geth and Lodestar services shares the same JWT secret for secure communication at the directory `/jwtsecret/secret` via a volume mounted to both containers.

## Getting started quickly

Run the command below to start the services

```bash
docker compose up
```

### Accessing the services

- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`
  - Default login: `admin` / `admin`
- Alertmanager: `http://localhost:9093`

## Logging

Logging is handled using Promtail and Loki:

- Promtail: Configured to collect logs from Docker containers via the docker socket and forward the collected logs to Loki. The config file is at `promtail/promtail.yml`. Modify as needed to add additional jobs.
- Loki: Receives log data from Promtail and stores it for querying. Configuration file can be found at `loki/loki.yml`

## Monitoring

Monitoring is managed with Prometheus, Node Exporter, and Grafana:

- Node Exporter: Exposes system metrics (e.g. CPU, memory etcetera) from the host machine and makes it available for a Prometheus job to scrape
- Prometheus: Configured at `prometheus/prometheus.yml` to scrape metrics from Node Exporter, as well as from the Ethereum Geth and Beacon node. Modify, as needed, with your own scrape jobs for more metrics. Alerting rules are also configured at `prometheus/alerting_rules.yml`
- Grafana: Visualizes the metrics from Prometheus through dashboards. Pre-configured dashboards for Geth, Lodestar, and Node Exporter are available at `grafana/dashboards/`. Additional dashboards can be added by placing JSON files in the grafana/dashboards/ directory. Prometheus and Loki are added to Grafana as a datasource via the `grafana/datasource.yml` file. Add more sources by modifying this file.

## Alerting

Alerting is configured using Alertmanager:

Alertmanager handles alerts sent by Prometheus and is configured to send these alerts to Slack. To receive Slack notifications, replace the placeholders `api_url` and `channel` in the `alertmanager/alertmanager.yml` file with your actual Slack webhook URL and channel where you wish to receive these alerts. Visit [Sending messages using incoming webhooks](https://api.slack.com/messaging/webhooks) for details.

Triggered alerts can also be viewed and managed through the Alertmanager web UI at <http://localhost:9093>.
