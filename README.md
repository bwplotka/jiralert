# JIRAlert 
[![Build Status](https://travis-ci.org/free/jiralert.svg)](https://travis-ci.org/free/jiralert) 
[![Go Report Card](https://goreportcard.com/badge/github.com/free/jiralert)](https://goreportcard.com/report/github.com/free/jiralert) 
[![GoDoc](https://godoc.org/github.com/free/jiralert?status.svg)](https://godoc.org/github.com/free/jiralert)
[![Slack](https://img.shields.io/badge/join%20slack-%23jiralert-brightgreen.svg)](https://join.slack.com/t/improbable-eng/shared_invite/enQtMzQ1ODcyMzQ5MjM4LWY5ZWZmNGM2ODc5MmViNmQ3ZTA3ZTY3NzQwOTBlMTkzZmIxZTIxODk0OWU3YjZhNWVlNDU3MDlkZGViZjhkMjc)

[Prometheus Alertmanager](https://github.com/prometheus/alertmanager) webhook receiver for [JIRA](https://www.atlassian.com/software/jira).

## Overview

JIRAlert implements Alertmanager's webhook HTTP API and connects to one or more JIRA instances to create highly configurable JIRA issues. One issue is created per distinct group key — as defined by the [`group_by`](https://prometheus.io/docs/alerting/configuration/#<route>) parameter of Alertmanager's `route` configuration section — but not closed when the alert is resolved. The expectation is that a human will look at the issue, take any necessary action, then close it.  If no human interaction is necessary then it should probably not alert in the first place.

If a corresponding JIRA issue already exists but is resolved, it is reopened. A JIRA transition must exist between the resolved state and the reopened state — as defined by `reopen_state` — or reopening will fail. Optionally a "won't fix" resolution — defined by `wont_fix_resolution` — may be defined: a JIRA issue with this resolution will not be reopened by JIRAlert.

## Usage

Get JIRAlert, either as a [packaged release](https://github.com/free/jiralert/releases) or build it yourself:

```
$ go get github.com/free/jiralert/cmd/jiralert
```

then run it from the command line:

```
$ jiralert
```

Use the `-help` flag to get help information.

```
$ jiralert -help
Usage of jiralert:
  -config string
      The JIRAlert configuration file (default "config/jiralert.yml")
  -listen-address string
      The address to listen on for HTTP requests. (default ":9097")
  [...]
```

## Testing

JIRAlert expects a JSON object from Alertmanager. The format of this JSON is described in the [Alertmanager documentation](https://prometheus.io/docs/alerting/configuration/#<webhook_config>) or, alternatively, in the [Alertmanager GoDoc](https://godoc.org/github.com/prometheus/alertmanager/template#Data).

To quickly test if JIRAlert is working you can run:

```bash
$ curl -H "Content-type: application/json" -X POST \
  -d '{"receiver": "jira-ab", "status": "firing", "alerts": [{"status": "firing", "labels": {"alertname": "TestAlert", "key": "value"} }], "groupLabels": {"alertname": "TestAlert"}}' \
  http://localhost:9097/alert
```

## Configuration

The configuration file is essentially a list of JiraAlert receivers plus defaults (in the form of a partially defined receiver); and a pointer to the template file.

You can find more docs in [the configuration itself](/pkg/config/config.go)

Each receiver must have:
* a unique name (matching the Alertmanager receiver name)
* JIRA API access fields (URL, username and password),
* handful of required issue fields (such as the JIRA project and issue summary), 
* some optional issue fields (e.g. priority) and a `fields` map for other (standard or custom) JIRA fields. 

Most of these may use [Go templating](https://golang.org/pkg/text/template/) to generate the actual field values based on the contents of the Alertmanager notification. 

The exact same data structures and functions as those defined in the [Alertmanager template reference](https://prometheus.io/docs/alerting/notifications/) are available in JIRAlert.

## Alertmanager configuration

To enable Alertmanager to talk to JIRAlert you need to configure a webhook in Alertmanager. You can do that by adding a webhook receiver to your Alertmanager configuration. 

```yaml
receivers:
- name: 'jira-ab'
  webhook_configs:
  - url: 'http://localhost:9097/alert'
    # JIRAlert ignores resolved alerts, avoid unnecessary noise
    send_resolved: false
```

## Profiling

JIRAlert imports [`net/http/pprof`](https://golang.org/pkg/net/http/pprof/) to expose runtime profiling data on the `/debug/pprof` endpoint. For example, to use the pprof tool to look at a 30-second CPU profile:

```bash
go tool pprof http://localhost:9097/debug/pprof/profile
```

To enable mutex and block profiling (i.e. `/debug/pprof/mutex` and `/debug/pprof/block`) run JIRAlert with the `DEBUG` environment variable set:

```bash
env DEBUG=1 ./jiralert
```

## Community

*Jiralert* is an open source project and we welcome new contributors and members 
of the community. Here are ways to get in touch with the community:

* Slack: [#jiralert](https://join.slack.com/t/improbable-eng/shared_invite/enQtMzQ1ODcyMzQ5MjM4LWY5ZWZmNGM2ODc5MmViNmQ3ZTA3ZTY3NzQwOTBlMTkzZmIxZTIxODk0OWU3YjZhNWVlNDU3MDlkZGViZjhkMjc)
* Issue Tracker: [GitHub Issues](https://github.com/free/jiralert/issues)

## License

JIRAlert is licensed under the [MIT License](https://github.com/free/jiralert/blob/master/LICENSE).

Copyright (c) 2017, Alin Sinpalean
