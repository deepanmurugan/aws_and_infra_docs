# Create Handler in Sensu Go

### Create slack workspace
```
Make sure you have valid workspace in slack, if not create one using this [link](https://slack.com/get-started#/create).

Add webhook info using https://YOUR WORKSPACE NAME HERE.slack.com/services/new/incoming-webhook and save settings.
```

### Create Handler-Slack
```
sensuctl handler create slack \
--type pipe \
--env-vars "SLACK_WEBHOOK_URL=https://hooks.slack.com/services/TXXXXXX/BXXXXX/XXXXXXXX" \
--command "sensu-slack-handler --channel '#sensu-go'" \
--runtime-assets sensu-slack-handler
```
Make sure you have sensu-slack-handler asset configured already. Update your slack webhook URL and channel in the above command.

### Update check config with the handler
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl check update local-memory-check
? Command: check-memory.rb -w 80 -c 90
? Interval: 60
? Cron: 
? Timeout: 0
? TTL: 
? Subscriptions: system
? Handlers: slack #Update handler name as slack here
? Runtime Assets:
? Publish: true
? Check Proxy Entity Name: 
? Check STDIN: false
? High Flap Threshold: 0
? Low Flap Threshold: 0
? Metric Format: none
? Metric Handlers: 
? Round Robin false
Updated
```
### Get check info to see if handler is updated in the check
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl check info local-memory-check --format wrapped-json
{
  "type": "CheckConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "local-memory-check",
    "namespace": "default",
    "created_by": "admin"
  },
  "spec": {
    "check_hooks": null,
    "command": "check-memory.rb -w 80 -c 90",
    "env_vars": null,
    "handlers": [
      "slack"
    ],
    "high_flap_threshold": 0,
    "interval": 60,
    "low_flap_threshold": 0,
    "output_metric_format": "",
    "output_metric_handlers": null,
    "proxy_entity_name": "",
    "publish": true,
    "round_robin": false,
    "runtime_assets": [
      "local/sensu-memory-checks",
      "local/sensu-ruby-runtime"
    ],
    "secrets": null,
    "stdin": false,
    "subdue": null,
    "subscriptions": [
      "system"
    ],
    "timeout": 0,
    "ttl": 0
  }
}
```
You will now see the sensu go alerts in the slack channel.
