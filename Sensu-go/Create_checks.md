# Create Sensu Go checks

You can use either sensuctl or using API to create checks. Let's see how we can create checks using sensuctl.

### Create check using sensuctl (inline)
```
sensuctl check create check-cpu \
--command 'check-cpu.rb -w 70 -c 90' \
--interval 60 \
--subscriptions system \
--runtime-assets "sensu-plugins/sensu-plugins-cpu-checks, sensu/sensu-ruby-runtime"
```

### Create check using sensuctl (json)
```
Create a file named check-cpu.json with below contents
{
  "type": "CheckConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "check-cpu",
    "namespace": "default",
    "created_by": "admin"
  },
  "spec": {
    "check_hooks": null,
    "command": "check-cpu.rb -w 70 -c 90",
    "env_vars": null,
    "handlers": [],
    "high_flap_threshold": 0,
    "interval": 60,
    "low_flap_threshold": 0,
    "output_metric_format": "",
    "output_metric_handlers": null,
    "proxy_entity_name": "",
    "publish": true,
    "round_robin": false,
    "runtime_assets": [
      "sensu-plugins/sensu-plugins-cpu-checks",
      "sensu/sensu-ruby-runtime"
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

root@ip-172-31-38-43:/home/ubuntu# sensuctl create -f check-cpu.json
```
### List the checks available
```
sensuctl check list
```

NOTE: Makes sure you give the asset name exactly how it is named when you create asset. Run sensuctl asset list, get asset name and update the same in the sensu check definition - runtime_assets section.

Parameter subscriptions is important here, this sensu check will run based on the subscriptions only. Sensu backend server will run this check on sensu agent which is also having this subscription named 'system'. When there is no matching subscriptions found this check will not be run on any agents.
