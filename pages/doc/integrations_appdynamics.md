---
title: AppDynamics Integration
keywords:
tags: [integrations]
sidebar: doc_sidebar
permalink: integrations_appdynamics.html
summary: Learn how to send data emitted by AppDynamics to Wavefront.
---
[AppDynamics](https://www.appdynamics.com/) is a popular APM solution for monitoring applications. It runs as a cloud service or on-premise. The Appdynamics integration captures minute level data and stores it in Wavefront without any loss of precision. This allows you to now create meaningful alerts using the powerful time series query language.
 
 
## Metric Collection
The [Wavefront collector](https://github.com/wavefrontHQ/wavefront-collector) collects AppDynamics metrics using the AppDynamics REST SDK. The Wavefront collector is a python script that runs periodically to collect metrics about the application, tiers and nodes. The Wavefront collector sends the metrics to the wavefront proxy which in turn forwards the metrics to the Wavefront backend.
 
## Instructions
 
1. Install the [Wavefront Collector](https://pypi.python.org/pypi/wavefront_collector): `pip install wavefront_collector`
1. Install [AppDynamicsREST](https://github.com/tradel/AppDynamicsREST): `pip install AppDynamicsREST`
1. Copy `appdynamics.conf` from `appdynamics-sample-configuration` to a directory of your choosing.
1. Open `appdynamics.conf` in an editor and update the `api`, `filter`, and `writer` sections.
1. Run the Wavefront Collector: `wf -c wavefront-collector.conf`
 
## Example AppDynamics conf File

```conf
[api]
; See https://docs.appdynamics.com/display/PRO41/Use+the+AppDynamics+REST+APIs
account = <todo>
username = <todo>
password = <todo>
controller_url = http://localhost:8081
debug = True
 
[filter]
; Get the application IDs from your controller user interface
; blacklist_regex should be in the form: A\|.*,B\|C\|.*,etc ...
application_ids = APP_ID_1,APP_ID_2,...
whitelist_regex =
blacklist_regex =
 
[options]
retrieve_BT_node_data = false
retrieve_error_node_data = false
retrieve_Application_Infrastructure_Performance_node_data = false
retrieve_EUM_AJAX_data = false
 
[writer]
; TODO: Change this host to match the host of your Wavefront proxy.
; TODO: change dry_run = False when ready to send data
host = 127.0.0.1
port = 2878
dry_run = True
```

The `options` field allows you to get granular metrics from node level data. However, note that you can increase the number of points sent to Wavefront when this option is turned on. We recommend that you run the collector with the option of `dry_run` set to `true`. This prints the metrics fetched from AppDynamics and give you the total count of metrics fetched in one run.

## Dashboard

Once you've installed AppDynamics, you can [deploy](dashboards_managing#deploying-a-dashboard) the [AppDynamics dashboard](https://github.com/wavefrontHQ/integrations/tree/master/telegraf/dashboards) to begin monitoring your server metrics from AppDynamics in Wavefront:

![db_appdynamics application](images/db_appdynamics_application.png)
![db_appdynamics backend](images/db_appdynamics_backend.png)

## Alerting

The biggest advantage of this integration is the flexible and dynamic alerting using the simple but powerful time series query language. AppDynamics alerting is based on standard deviation or a value exceeding a threshold.

![alert_appdynamics](images/alert_appdynamics.png)

But what if your data does not follow the standard distribution or you want to do math on the incoming data such as take a ratio of two metrics? With Wavefront you can use the query language to model the shape of your data or do a ratio and then alert on the derived metrics.

![edit_alert_appdynamics](images/edit_alert_appdynamics.png)

{% include links.html %}