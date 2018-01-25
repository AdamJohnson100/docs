---
title: Monitoring Wavefront
tags: [administration, dashboards]
sidebar: doc_sidebar
permalink: wavefront_monitoring.html
summary: Learn how to monitor and troubleshoot the health of your Wavefront instance.
---

If system performance seems to be deteriorating, you can examine your Wavefront instance and Wavefront proxy with the Wavefront system dashboard, and look at internal metrics to investigate the problem.

This page discusses monitoring your Wavefront instance. See [Monitoring Wavefront Proxies](monitoring_proxies.html) for details on investigating proxy issues.

## Wavefront Internal Metrics Overview

Wavefront collects several categories of internal metrics. These categories have the following prefixes:

- `~alert*` - set of metrics that allows you to examine the effect of alerts on your Wavefront instance. See [Troubleshooting Your Wavefront Instance with Internal Metrics](wavefront_monitoring.html#troubleshooting-your-wavefront-instance-with-internal-metrics)
- `~collector` - metrics processed at the collector gateway to the Wavefront instance.
- `~metric` - total unique sources and metrics.  You can compute the rate of metric creation from each source. [Troubleshooting Your Wavefront Instance with Internal Metrics](wavefront_monitoring.html#troubleshooting-your-wavefront-instance-with-internal-metrics) discusses a set of `~metric.new*` metrics.
- `~proxy` - metric rate received and sent from each Wavefront proxy, blocked and rejected metric rates, buffer metrics, and JVM stats of the proxy. Also includes counts of metrics affected by the proxy preprocessor.
  {% include note.html content="Proxy metrics historically had the prefix `~agent` and queries support both `~proxy` and `~agent`. Query errors still refer to the `~agent` prefix. For example - `No metrics matching - [~agent.points.*.received]`." %}
  See [Monitoring Wavefront Proxies](monitoring_proxies.html).
- `~wavefront` - set of gauges that track metrics about your use of Wavefront.

If you have an [AWS integration](integrations_aws_metrics.html), metrics with the following prefix are available:

- `~externalservices` - metric rates, API requests, and events from AWS CloudWatch, AWS CloudTrail, and AWS Metrics+.

## Charts in the Wavefront Usage Integration Dashboard

The [Wavefront Usage integration](integrations.html#in-product-integrations) provides the Wavefront System Usage dashboard that displays metrics that help you find reasons for system slowdown. You can examine many aspects or your Wavefront Instance. We'll look at  the following sections here:
* Overall Data Rate
* Wavefront Stats
* AWS Integration
* Ingestion Rate by Source

See [Monitoring Wavefront Proxies](wavefront_monitoring_proxies.html) for details on the following sections:
* Proxy Health
* Proxy Troubleshooting

### Overall Data Rate

The Overall Data Rate section shows the overall point rate being processed by the Wavefront servers.

![overall_section](images/overall_section.png)

These charts use the following metrics:

- **Data Ingestion Rate** - `~collector.points.reported`, `~externalservices.cloudwatch.points`, and `~externalservices.ec2.points`, counter metrics the per second rate at which new data points are being ingested into Wavefront. The AWS metrics are broken out in [AWS Integration](#aws-integration-metrics).
- **Data Scan Rate** - `~query.summaries_scanned`, the per second rate at which data points are being queried out of Wavefront through dashboards, alerts, custom charts, or API calls.


### Wavefront Stats

Charts that track the number of Wavefront users during various time windows, number of dashboards and alerts, and information about the types of alerts.

![wavefront metrics](images/wavefront_metrics.png)

### AWS Integration

If you have an [AWS integration](integrations_aws_metrics.html) and are ingesting AWS CloudWatch, CloudTrail, and API Metrics+ metrics into Wavefront, this section monitors the count of CloudWatch requests, API requests, the point rate, and events coming in from your integration.

![aws_metric_sections](images/aws_metric_sections.png)

The available metrics are:

- `~externalservices.cloudwatch.api-requests` - number of CloudWatch API requests
- `~externalservices.cloudwatch.points`- number of CloudWatch metrics returned
- `~externalservices.ec2.points` - number of AWS Metrics+ metrics returned
- `~externalservices.cloudtrail.events` - number of CloudTrail events returned
- `~externalservices.cloudwatch-cycle-timer` - time in milliseconds CloudWatch requests take to complete

### Ingest Rate by Source

This section gives insight into the shape of your data. It shows the total number of sources reporting. It also monitors the rate of metrics creation and breaks it down by source.

![point_rate breakdown](images/point_rate_breakdown.png)

The metrics used in this section are:

- `~metric.counter` - number of metrics being collected. It can be broken down by the sources sending the metrics.

## Using Internal Metrics to Optimize Performance

A small set of internal metrics can help you optimize performance. This section highlights some things to look for - the exact steps depend on how you're using Wavefront and on the characteristics of your environment. The following internal metrics were added to Wavefront in the 2017.52 release based on suggestions from our customer support engineers.

* `~query.requests`
* `~metric.new_host_ids`
* `~metric.new_metric_ids`
* `~metric.new_string_ids`
* `~alert.query_time.{alert_id}`
* `~alert.query_points.{alert_id}`
* `~alert.checking_frequency.{id}`

Here's one easy way to see this new information:
1. Select **Integrations** and click the Wavefront Usage integration.
2. Select **Dashboard**.
3. Click the pencil icon and select **Clone**.
4. Add charts for the metrics that you're interested in.

### Fine-Tuning Alerts

The `~alert` metrics allow you to examine your alerts and understand which alerts impact performance. After you find out how much load a query is putting on the system, you can potentially refine the alert and improve performance.
* `~alert.query_points` shows the details of the points scanned by each alert.
* `~alert.query_time` shows details for the amount of time it takes to run the alert query.
* `~alert.checking_frequency` helps you find alerts that are checking too frequently. For each alert, the alert checking frequency should be greater or equal to query time.

For example, you can set up an alert that monitors existing alerts that have high points scanned rates. You can then catch badly written alerts and tune them to improve performance.

See [Alert Dependencies](/alerts_dependencies.html) for additional information on fine-tuning your alerts using internal metrics.

### Understanding System Activity

The three `~metric.new_*` internal metrics allow you to discover if a recent change to the system might have caused the problem. These metrics can show you if Wavefront recently received points that don't fit the usual pattern of the metrics that Wavefront received from you. For example, assume you just used the Kubernetics integration to add a cluster to your Wavefront instance. The integration will start sending data from all hosts in the cluster. If you create point tags, they will also be sent for each host, potentially creating a bottleneck.

Each metric includes the metric name, customer, any tags, and the source or host. The three internal metrics allow you to find out information about 3 aspects of the metric.
* `~metric.new_metric_ids` shows metrics that Wavefront hasn't seen before in the metric namespace.
* `~metric.new_string_ids` shows point tags that Wavefront hasn't seen before, as strings.
* `~metric.new_host_ids` shows hosts, that is, the sources for the metrics, that Wavefront hasn't seen before.


### Find Users Who Caused Bottlenecks

 `~query.requests` returns information about queries and the associated user. It helps you examine whether one of your users stands out as the person who might be causing the performance problem. Often, new users unintentionally send many queries to Wavefront, especially if they use the API for the queries. The results can become difficult to interpret, and system performance might suffer.
