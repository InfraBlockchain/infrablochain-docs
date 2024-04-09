---
title: Node Metric Monitoring
description: Capture and visualize information about Substrate nodes using observability tools.
keywords:
  - observability
  - metrics
  - node operations
---

Substrate exposes metrics for network operations. For example, you can collect information about the number of peers connected to a node, the amount of memory used by the node, and the number of blocks generated. To capture and visualize the metrics exposed by a Substrate node, you can configure and use tools like [Prometheus](https://prometheus.io/) and [Grafana](https://grafana.com/). 

This tutorial demonstrates how to use Prometheus to collect data samples and Grafana to create graphs and dashboards visualizing node metrics using the collected data samples.

The following diagram provides a simplified overview of the interaction between Substrate, Prometheus, and Grafana in visualizing node metrics:

![Visualizing Node Metrics with Prometheus and Grafana](/media/images/docs/infrablockchain/tutorials/node-metrics.png)

## Prerequisites

Before getting started, make sure you have:

- Installed Rust and the Rust toolchain to set up your Substrate development environment. See the [installation guide](/install/) for instructions.

- Completed some of the previous tutorials, including [Building a Local Blockchain](/tutorials/build-a-blockchain/build-local-blockchain/) and [Simulating a Network](/tutorials/build-a-blockchain/simulate-network/).

## Tutorial Objectives

By completing this tutorial, you will be able to:

- Install Prometheus and Grafana.
- Configure Prometheus to capture time-series data from a Substrate node.
- Configure Grafana to visualize the collected node metrics using the Prometheus endpoint.

## Installing Prometheus and Grafana

For testing and demo purposes, it is recommended to download the compiled `bin` programs of Prometheus and Grafana instead of building the tools yourself or using Docker images. Use the following links to download the appropriate pre-compiled binaries for your architecture. This tutorial assumes that you will be using the compiled binaries in your working directory.

To install the tools for this tutorial, follow these steps:

1. Open a browser on your computer.

2. Go to the [Prometheus download page](https://prometheus.io/download/) and download the appropriate pre-compiled binary.

3. Open a terminal shell on your computer and execute a command similar to the following to extract the contents from the downloaded file:

   For example, on macOS, you can execute a command similar to the following:

   ```text
   gunzip prometheus-2.38.0.darwin-amd64.tar.gz && tar -xvf prometheus-2.38.0.darwin-amd64.tar
   ```

4. Go to the [Grafana OSS download page](https://grafana.com/grafana/download?edition=oss).

5. Choose the appropriate pre-compiled binary for your architecture.

6. Open a terminal shell on your computer and execute the command specific to your architecture to install Grafana.

   For example, on macOS with Homebrew installed, you can execute the following command:

   ```text
   brew update
   brew install grafana
   ```

## Starting the Substrate Node

Substrate exposes metrics in the [Prometheus exposition format](https://prometheus.io/docs/concepts/data_model/) on port `9615`. You can change the port using the `--prometheus-port` command-line option and configure it to be accessible on interfaces other than localhost using the `--prometheus-external` command-line option. For simplicity, this tutorial assumes that both the Substrate node, Prometheus instance, and Grafana service are running locally using the default ports.

1. Open a terminal shell on your computer.

2. If necessary, navigate to the root of the node template directory. You can navigate using the following command:

   ```bash
   cd substrate-node-template
   ```

3. Start the node template in development mode by executing the following command:

   ```bash
   ./target/release/node-template --dev
   ```

## Configuring the Prometheus Endpoint

When you extract the Prometheus download, the extracted directory contains a `prometheus.yml` configuration file. You can modify this file or create a custom configuration file to configure Prometheus to scrape data from the default Prometheus port endpoint (port `9615`) where the Substrate node exposes its metrics. You can also change the port by using the `--prometheus-port <port number>` command-line option.

To add the Substrate exposed endpoint to the Prometheus target list, follow these steps:

1. Open a new terminal shell on your computer.

2. Navigate to the root of the Prometheus working directory.

3. Open the `prometheus.yml` configuration file in a text editor.

4. Add `substrate_node` as a scrape target to the `scrape_config` endpoint.

   For example, add a section similar to the following:

   ```yml
    # A scrape configuration containing exactly one endpoint to scrape:
    scrape_configs:
      - job_name: "substrate_node"

        scrape_interval: 5s

        static_configs:
          - targets: ["localhost:9615"]
   ```

   This configuration overrides the global defaults for scraping targets for the `substrate_node` job. It is important to set a `scrape_interval` value smaller than the block time to ensure that there is a data point for every block. For example, the Polkadot block time is 6 seconds, so you would set the `scrape_interval` to 5 seconds.

1. Start the Prometheus instance with the modified `prometheus.yml` configuration file.

   For example, if you are currently in the Prometheus working directory and using the default configuration file name, you can start Prometheus by executing the following command:

   ```bash
   ./prometheus --config.file prometheus.yml
   ```

   Leave this process running.

2. Open a new terminal shell and execute the following command to retrieve the metrics for the Substrate node:

   ```bash
   curl localhost:9615/metrics
   ```

   This command returns output similar to the following (only showing some examples):

   ```text
   # HELP substrate_block_height Block height info of the chain
   # TYPE substrate_block_height gauge
   substrate_block_height{status="best",chain="dev"} 16
   substrate_block_height{status="finalized",chain="dev"} 14
   substrate_block_height{status="sync_target",chain="dev"} 16
   # HELP substrate_block_verification_and_import_time Time taken to verify and import blocks
   # TYPE substrate_block_verification_and_import_time histogram
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.005"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.01"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.025"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.05"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.25"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="2.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="10"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="+Inf"} 0
   substrate_block_verification_and_import_time_sum{chain="dev"} 0
   substrate_block_verification_and_import_time_count{chain="dev"} 0
   # HELP substrate_build_info A metric with a constant '1' value labeled by name, version
   # TYPE substrate_build_info gauge
   substrate_build_info{name="ruddy-afternoon-1788",version="4.0.0-dev-6a8b2b12371",chain="dev"} 1
   # HELP substrate_database_cache_bytes RocksDB cache size in bytes
   # TYPE substrate_database_cache_bytes gauge
   substrate_database_cache_bytes{chain="dev"} 0
   # HELP substrate_finality_grandpa_precommits_total Total number of GRANDPA precommits cast locally.
   # TYPE substrate_finality_grandpa_precommits_total counter
   substrate_finality_grandpa_precommits_total{chain="dev"} 76
   # HELP substrate_finality_grandpa_prevotes_total Total number of GRANDPA prevotes cast locally.
   # TYPE substrate_finality_grandpa_prevotes_total counter
   substrate_finality_grandpa_prevotes_total{chain="dev"} 77
   # HELP substrate_finality_grandpa_round Highest completed GRANDPA round.
   # TYPE substrate_finality_grandpa_round gauge
   substrate_finality_grandpa_round{chain="dev"} 76
   ...
   ```

   Alternatively, you can open the same endpoint in your browser to see all the available metric data. For example, if you are using the default Prometheus port, open [`http://localhost:9615/metrics`](http://localhost:9615/metrics) in your browser.

## Configuring the Grafana Data Source

After installing Grafana using the appropriate command for your operating system and starting the service on your local machine, you can access it in your browser.
The command used to start the service depends on your local system architecture and package manager.
For example, if you are using macOS and Homebrew, you can start Grafana by running the following command:

```bash
brew services start grafana
```

For information on how to start Grafana on other operating systems, see the corresponding [Grafana](https://grafana.com/docs/grafana/v9.0/) documentation.

After starting Grafana, you can navigate to Grafana from your browser.

1. Open a browser and go to the Grafana URL, which uses the port Grafana is configured to use.

   By default, Grafana uses http://localhost:3000, but if you configured a different host or port, use that value.

2. Log in using the default `admin` username and password `admin`, then click **Log in**.

3. On the welcome page, click **Configuration**, then click **Data Sources** under the **Configuration** menu.

4. Click **Prometheus** to configure Prometheus as the data source for Substrate node metrics.

   If both the Substrate node and Prometheus instance are running, configure Grafana to find Prometheus at the default port `http://localhost:9090` or the custom port information you configured. 
   
   Do not specify the Prometheus port you set in the `prometheus.yml` file. That port is where the node publishes data.

2. Click **Save & Test** to verify that the data source is configured correctly.

   If the data source is working, you are ready to configure a dashboard to display node metrics.

## Importing the Template Dashboard

If you need a starting point, you can import a basic dashboard that provides information about the node. You can download the [Substrate Dashboard Template](/assets/tutorials/monitor-node/substrate-node-template-metrics.json) to get started.

To import the dashboard template, follow these steps:

1. On the Grafana welcome page, click **Dashboards**.

1. Click **Dashboards** in the left navigation, then select **Browse**.

2. Click **New** for the search options, then select **Import**.



3. Copy the [Substrate Dashboard Template](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/monitor-node/substrate-node-template-metrics.json) and paste it into the **Import via panel json** text box.

4. Click on **Load**.

5. Review and modify the name, folder, and unique identifier of the dashboard if necessary.

6. Select **Prometheus (default)** and then click on **Import**.

   ![Substrate Dashboard Template](/media/images/docs/tutorials/monitor-node-metrics/grafana-template-dashboard.png)

   The [Substrate Dashboard Template](https://grafana.com/grafana/dashboards/13759/) can be used for Substrate-based chains and can also be downloaded from the Grafana Labs Dashboard Gallery.

   If you want to create a custom dashboard, refer to the [Grafana section of the Prometheus documentation](https://prometheus.io/docs/visualization/grafana/).

   If you have created a custom dashboard, consider uploading it to the [Grafana Dashboards](https://grafana.com/grafana/dashboards).
   
    You can also list it in the [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) repository to notify the Substrate builder community.

## Moving on to the next steps

- [Adding Trusted Nodes](/tutorials/build-a-blockchain/add-trusted-nodes)
- [Node Monitoring](https://wiki.polkadot.network/docs/en/maintain-guides-how-to-monitor-your-node)
- [Substrate Prometheus Exporter](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/utils/prometheus)
- [Polkadot Network Dashboard](https://github.com/w3f/polkadot-dashboard)