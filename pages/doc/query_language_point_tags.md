---
title: Point Tags in Queries
keywords: query language, point tags
tags: [query language]
sidebar: doc_sidebar
permalink: query_language_point_tags.html
summary: Learn how to use point tags in Wavefront Query Language queries.
---
Point tags are key-value pairs (strings) that are associated with a point. Point tags provide additional context for your data and allow you to fine-tune your queries so the output shows just what you need.

**Note:** If point tag values are blank or empty, errors can result. Point tag values can be zero.

## Point Tag Basics

You can use point tags, for example, to label a point's datacenter, version, etc. and can then group by datacenter or version.

You add point tags using [Wavefront proxy preprocessor rules](proxies_preprocessor_rules.html).

**Note:** For best performance, keep the number of distinct time series per metric and host to under 1000. See Best Practices for Point Tags below. 

The [AWS cloud integration](integrations_aws_metrics.html#wavefront-point-tags) generates point tags automatically to help you filter those metrics.


Suppose you send the following points, all from a single source `cache1`, over a 5 minute period:

```r
test.request.latency 30 1394740020 source=cache1 clientService="API"
test.request.latency 25 1394740080 source=cache1 clientService="API"
test.request.latency 34 1394740140 source=cache1 clientService="API"
test.request.latency 37 1394740200 source=cache1 clientService="API"
test.request.latency 16 1394740240 source=cache1 clientService="API"
test.request.latency 45 1394740020 source=cache1 clientService="web"
test.request.latency 53 1394740080 source=cache1 clientService="web"
test.request.latency 25 1394740140 source=cache1 clientService="web"
test.request.latency 60 1394740200 source=cache1 clientService="web"
test.request.latency 30 1394740240 source=cache1 clientService="web"
```
The first 5 points are request latencies from API calls, and the last 5 points are request latencies from web calls. If you query `ts(test.request.latency`), you'll see two lines:

![Two lines](images/two_lines.png)

The orange line is associated with the point tag `clientService=web`, while the blue line is associated with the point tag `clientService=API`.

If you have multiple point tags on a point, you'll see all the point tags. For this example, the points in the green line contain three point tags: `clientService`, `clientApp`, and `hw`. Note that they are all visible in the legend:

![Three lines](images/three_lines.png)

## Filtering Queries Using Point Tags

To see the request latencies that have the `clientService="API"` point tag use the query:

```r
ts(test.request.latency, clientService="API")
```

![One point tag](images/one_point_tag.png)

The query returns one line because all points share the same point tag key-value pairs.

Suppose 5 points have the `clientService="batch"` point tag and other point tags:

```r
test.request.latency 45 1394740020 source=cache1 clientService="batch" clientApp="dailyReport" hw="vm045.wavefront.com"
test.request.latency 47 1394740080 source=cache1 clientService="batch" clientApp="dailyReport" hw="vm045.wavefront.com"
test.request.latency 44 1394740140 source=cache1 clientService="batch" clientApp="dailyReport"
test.request.latency 25 1394740200 source=cache1 clientService="batch" clientApp="hourlyReport"
test.request.latency 52 1394740240 source=cache1 clientService="batch" clientApp="hourlyReport"
```

You can query for all 5 points using `ts(test.request.latency, clientService="batch")`:

![Three point tags](images/three_point_tags.png)

The query retrieves all of the points, but they aren't charted as a single line because they don't represent the same set of point tag key-value pairs.

Finally, you can add another point tag to the query to further filter:

```r
ts(test.request.latency, clientService="batch" and clientApp="hourlyReport")
```

![Both point tags](images/both_point_tags.png)

Now only a single a series that matches both point tags displays; all of the other series (with different point tags) are filtered out.

Point tags offer you a powerful way of labeling your data so that you can slice and dice it in almost any way you can imagine.

## Best Practices for Point Tags

Wavefront recommends that you keep the number of distinct time series per metric and host to under 1000. Whether a time series is distinct depends on the combination of the point tag keys and the point tag values.

For example, assume a metric `cpu.idle` and a host `web1`.  If you use that metric and host with the point tags `env=prod` and `datacenter=atl`, a new time series results. If you use `env=dev` and `datacenter=atl`, another distinct time series results.

Using point tags to store highly variable data such as timestamps, login emails, or web session IDs will eventually cause performance issues when your data are queried. That is also true if you specify a time that results in many time series being retrieved. For example `timestamp=<now>` or even `monthofyear=11` can exceed the limit. In contrast, `dayofweek=monday` or `monthofyear=jan` are acceptable.

See [Series Matching](query_language_series_matching.html) for more info on:

* Series matching with point tags.
* Series matching with point tags and the `by`construct.
