# sa-prometheus


[![Build Status](https://travis-ci.org/softasap/sa-prometheus.svg?branch=master)](https://travis-ci.org/softasap/sa-prometheus)


# Usage:

## Architecture overview

![](docs/architecture.svg)

Simple:

```YAML

- {
    role: "sa-prometheus"
  }


```

Advanced:

```YAML

vars:

  some_custom_properties:
    - {regexp: "^;allow_sign_up =(.*)", line: "allow_sign_up=false"}


roles:
  - {
      role: "sa-prometheus",

      option_install_go: true,
      go_version: 1.6,


      prometheus_user:   prometheus,
      prometheus_group:  prometheus,

      prometheus_base_dir: /opt/prometheus,
      prometheus_data_dir: "{{ prometheus_base_dir }}/data",

      prometheus_version:                 2.0.0,
      prometheus_node_exporter_version:   0.15.2,
      prometheus_alertmanager_version:    0.11.0,

      prometheus_config_path:          /etc/prometheus
    }


```


Best cookied with Grafana (example included in box example , see related article on  https://www.digitalocean.com/community/tutorials/how-to-add-a-prometheus-dashboard-to-grafana)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_011.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_012.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_013.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_014.png)


Further reading:


https://www.digitalocean.com/community/tutorials/how-to-query-prometheus-on-ubuntu-14-04-part-1

https://www.digitalocean.com/community/tutorials/how-to-query-prometheus-on-ubuntu-14-04-part-2


Misc hints
----------

Top 10 metrics stored in db at a moment

```
topk(10, count by (__name__)({__name__=~".+"}))
```

## Typical monitoring constructs

using the default exporters installed together with `sa_prometheus`

### Portals availability

with help of the blackbox exporter

```yaml
  - job_name: 'blackbox_web'
    scrape_interval: 120s
    scrape_timeout:  10s
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets/portals/*.json'
        - '/etc/prometheus/targets/portals/*.yaml'
        - '/etc/prometheus/targets/portals/*.yml'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.

```

and in target/portals: multiple files `google.com.yml`

```yaml
---
- targets:
  - https://google.com
  labels:
    instance: google.com
```

corresponding rules/portal_alerts.yaml

```yaml
groups:
- name: portals.rules
  rules:
  - alert: google_com
    expr: probe_http_status_code{instance="google.com"} != 200
    for: 2m
    labels:
      severity: critical
    annotations:
      description: The google.com site is returning status codes other than 200,
        this usually means that the site down
      title: google.com is returning errors for 2m
```

### Validating ssl certificates

with help of ssl exporter

```yaml
  - job_name: 'ssl'
    metrics_path: /probe
    static_configs:
      - targets:
        - https://google.com/
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9219  # SSL exporter.
```

corresponding rules/ssl_alerts.yaml

```yaml

groups:
- name: ssl_alerts.rules
  rules:
  - alert: Certificate expired
    expr: probe_success{job="ssl"} == 0
    labels:
      severity: critical
    annotations:
      description: '[{{ $labels.env }}]{{ $labels.instance }} {{ $labels.name }} of job {{ $labels.job }} seems have ssl cert expired'
      summary: 'Portal {{ $labels.instance }} has possible ssl certificate issue issue'
    for: 10m

  - alert: Valid SSL connection cannot be established
    expr: ssl_tls_connect_success == 0
    labels:
      severity: warning
    annotations:
      description: '[{{ $labels.env }}]{{ $labels.instance }} {{ $labels.name }} of job {{ $labels.job }} seems have ssl issues'
      summary: 'Portal {{ $labels.instance }} has possible ssl issues'
    for: 10m

  - alert: Certificate expires soon
    expr: ((ssl_cert_not_after - time() < 86400 * 2) * on (instance,issuer_cn,serial_no) group_left (dnsnames) ssl_cert_subject_alternative_dnsnames) * on (instance,issuer_cn,serial_no) group_left (subject_cn) ssl_cert_subject_common_name > 0
    labels:
      severity: warn
    annotations:
      title: Certificate expires within next 2 days
      description: |
        Consider checking soon {{ $labels.instance }} portal {{ $labels.dnsnames }}
    for: 15m


```

### Basic alert manager config

In case, if you have installed prometheus alert manager with https://github.com/softasap/sa-prometheus-alertmanager

your minimal `config.yml` based on default example would be mostly inserting few lines of credentials

```yaml

global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'system@fqdn'
  smtp_auth_username: 'system@fqdn'
  smtp_auth_password: 'password'

templates:
- '/etc/prometheus_alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.                                                                                                                                 [53/539]
route:
  # The labels by which incoming alerts are grouped together. For example,
  # multiple alerts coming in for cluster=A and alertname=LatencyHigh would
  # be batched into a single group.
  group_by: ['alertname', 'cluster', 'service']

  # When a new group of alerts is created by an incoming alert, wait at
  # least 'group_wait' to send the initial notification.
  # This way ensures that you get multiple alerts for the same group that start
  # firing shortly after another are batched together on the first
  # notification.
  group_wait: 30s

  # When the first notification was sent, wait 'group_interval' to send a batch
  # of new alerts that started firing for that group.
  group_interval: 5m

  # If an alert has successfully been sent, wait 'repeat_interval' to
  # resend them.
  repeat_interval: 3h

  # A default receiver
  receiver: slack-channel

  # All the above attributes are inherited by all child routes and can
  # overwritten on each.

  # The child route trees.

  routes:
  # This routes performs a regular expression match on alert labels to
  # catch alerts that are related to a list of services.
  - match_re:
      service: ^(foo1|foo2|baz)$
    receiver: slack-channel
    # The service has a sub-route for critical alerts, any alerts
    # that do not match, i.e. severity != critical, fall-back to the
    # parent node and are sent to 'team-X-mails'
    routes:
    - match:
        severity: critical
      receiver: slack-channel
  - match:
      service: files
    receiver: slack-channel

    routes:
    - match:
        severity: critical
      receiver: slack-channel

  # This route handles all alerts coming from a database service. If there's
  # no team to handle it, it defaults to the DB team.
  - match:
      service: database
    receiver: slack-channel
    # Also group alerts by affected database.
    group_by: [alertname, cluster, database]
    routes:
    - match:
        owner: team-X
      receiver: slack-channel
    - match:
        owner: team-Y
      receiver: slack-channel

# Inhibition rules allow to mute a set of alerts given that another alert is
# firing.
# We use this to mute any warning-level notifications if the same alert is
# already critical.
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  # Apply inhibition if the alertname is the same.
  equal: ['alertname', 'cluster', 'service']


receivers:
- name: slack-channel
  email_configs:
  - to: someone@gmail.com
  slack_configs:
  - channel: #notifications
    send_resolved: true
    api_url: https://hooks.slack.com/services/XXXX
    icon_url: https://avatars3.githubusercontent.com/u/4029521
    title: '{{ template "custom_title" . }}'
    text: '{{ template "custom_slack_message" . }}'
  - channel: "system_alerts"
    send_resolved: true
    api_url: https://hooks.slack.com/services/YYYY
    icon_url: https://avatars3.githubusercontent.com/u/4029521
    title: '{{ template "custom_title" . }}'
    text: '{{ template "custom_slack_message" . }}'

templates:
- '/etc/prometheus_alertmanager/template/*.tmpl'

```

usage with ansible-galaxy workflow
----------------------------------

If you installed the `sa-prometheus` role using the command


`
   ansible-galaxy install softasap.sa-prometheus
`

the role will be available in the folder `library/softasap.sa-prometheus`
Please adjust the path accordingly.

```YAML

     - {
         role: "softasap.sa-prometheus"
       }

```


Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.

Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html
