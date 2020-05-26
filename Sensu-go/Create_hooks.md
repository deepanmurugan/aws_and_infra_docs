# Create hook for a sensu check

Hooks are reusable commands the agent executes in response to a check result before creating a monitoring event. You can create, manage, and reuse hooks independently of checks. Hooks enrich monitoring event context by gathering relevant information based on the exit status code of a check (ex: 1). Hook commands can also receive JSON serialized Sensu client data via STDIN.

Here we are going to create a check to monitor supervisor process and start the supervisor process automatically when a critical event occurs.

Use [check-process asset] to download and configure. Check previous section if you want to configure and use assets.

### Create a hook to run command
```
Create a config file named start-supervisor-hook.json and update the below content.

{
  "type": "HookConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "start-supervisor",
    "namespace": "default",
    "labels": {
      "sensu.io/managed_by": "sensuctl"
    },
    "created_by": "admin"
  },
  "spec": {
    "command": "sudo service supervisor start",
    "runtime_assets": null,
    "stdin": false,
    "timeout": 60
  }
}

sensuctl create -f start-supervisor-hook.json
```
### List hook
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl hook list
        Name                    Command              Timeout   Stdin?  
 ────────────────── ─────────────────────────────── ───────── ──────── 
  start-supervisor   sudo service supervisor start        60   false   
```
### Create check with start-supervisor hook
```
Create a file named supervisor-process-check.json
{
  "type": "CheckConfig",
  "api_version": "core/v2",
  "metadata": {
    "name": "local-process-supervisor-check",
    "namespace": "default",
    "labels": {
      "sensu.io/managed_by": "sensuctl"
    },
    "created_by": "admin"
  },
  "spec": {
    "check_hooks": [
      {
        "critical": [
          "start-supervisor"
        ]
      }
    ],
    "command": "check-process.rb -p supervisor",
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
      "local/sensu-process-checks",
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

sensuctl create -f supervisor-process-check.json
```
Here the check_hooks section will trigger start-supervisor hook whenever 'critical' event occurs.

You may need to add permission to sensu user to become sudo since we are using sudo with the command. Now whenever supervisor is  down the hook will get triggered automatically and start supervisor process.

NOTE: Hooks will get executed automatically whenever an event occurs.
