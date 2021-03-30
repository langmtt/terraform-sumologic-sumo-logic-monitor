# sumologic-postgresql-monitors

This script configures Sumo Logic Monitors for PostgreSQL using Terraform modules.

## Getting Started

### Requirements

* [Terraform 0.13+](https://www.terraform.io/downloads.html)
* [Sumo Logic Provider](https://registry.terraform.io/providers/SumoLogic/sumologic/latest/docs)
* [Sumo Logic Monitors Module](https://registry.terraform.io/modules/SumoLogic/sumo-logic-monitor/sumologic/latest)


The `versions.tf` file lists the requirements.
Running `terraform init` will automatically install the required providers and the required modules.


### Sumo Logic Provider

Configure the Sumo Logic Access Key, Access Id and Deployment in the file `sumologic_provider_and_monitors_folder`.

```shell
provider "sumologic" {
  access_id   = "<SUMOLOGIC ACCESS ID>"
  access_key  = "<SUMOLOGIC ACCESS KEY>"
  environment = "<SUMOLOGIC DEPLOYMENT>"
}
```
You can also define these values in `terraform.tfvars`.

### Installation Steps

* Make sure that `Terraform 0.13+` is installed.
* Run `terraform init`.
* Configure `Sumo Logic Provider` in the file `sumologic_provider_and_monitors_folder.tf` as explained in the previous step.
* Optionally, update the monitor folder name in the file `sumologic_provider_and_monitors_folder.tf`.
* Configure Email and Connection notifications in the file `postgresql_notifications.auto.tfvars`.
* Run `terraform plan` to view the resources which will be created/modified by Terraform.
* Run `terraform apply`.

### Sample Email and Connection Notification Configuration File Values

```shell
group_notifications = true
connection_notifications = [
    {
      connection_type       = "PagerDuty",
      connection_id         = "<CONNECTION_ID>",
      payload_override      = "{\"service_key\": \"your_pagerduty_api_integration_key\",\"event_type\": \"trigger\",\"description\": \"Alert: Triggered {{TriggerType}} for Monitor {{Name}}\",\"client\": \"Sumo Logic\",\"client_url\": \"{{QueryUrl}}\"}",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    },
    {
      connection_type       = "Webhook",
      connection_id         = "<CONNECTION_ID>",
      payload_override      = "",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    }
  ]

email_notifications = [
    {
      connection_type       = "Email",
      recipients            = ["abc@example.com"],
      subject               = "Monitor Alert: {{TriggerType}} on {{Name}}",
      time_zone             = "PST",
      message_body          = "Triggered {{TriggerType}} Alert on {{Name}}: {{QueryURL}}",
      run_for_trigger_types = ["Critical", "ResolvedCritical"]
    }
  ]
}
```

### Monitors Created

| Type (Metrics/)|Name|Description|Trigger Type (Critical / Warning / MissingData)|
|---|---|---|---|
Logs|PostgreSQL - Instance Down|This alert fires when the Postgres instance is down|Critical
Metrics |   PostgreSQL - TooManyConnections| This alert fires when we detect that a PostgreSQL instance has too many (90% of allowed) connections) |Critical
Logs|PostgreSQL - SlowQueries| This alert fires when we detect that the PostgreSQL instance is executing slow queries|  Warning
Metrics |   PostgreSQL - Commit Rate Low|  This alert fires when we detect that Postgres seems to be processing very few transactions. |Critical
Logs|PostgreSQL - High Rate of Statement Timeout| This alert fires when we detect Postgres transactions show a high rate of statement timeouts  |Critical
Metrics |   PostgreSQL - High Rate Deadlock| This alert fires when we detect deadlocks in a Postgres instance  |Critical
Metrics |   PostgreSQL - High Replication Lag| This alert fires when we detect that the Postgres Replication lag (in bytes) is high. |Critical
Metrics | pg_stat_ssl View  PostgreSQL - SSL Compression| Active This alert fires when we detect database connections with SSL compression are enabled. This may add significant jitter in replication delay. Replicas should turn off SSL compression via `sslcompression=0` in `recovery.conf` |Critical
Metrics |   PostgreSQL - Too Many Locks Acquired|  This alert fires when we detect that there are too many locks acquired on the database. If this alert happens frequently, you may need to increase the postgres setting max_locks_per_transaction.  |Critical
Logs|PostgreSQL - Access from Highly Malicious Sources| This alert will fire when a Postgres instance is accessed from known malicious IP addresses.  |Critical