# Sensu Go Filters

Filters are used to filter out noises from sensu alerts. Below filter named 'hourly' will send sensu alert only when the first occurence of the event and after every hour.

### Create hourly.json using below content
```
{
  "type": "EventFilter",
  "api_version": "core/v2",
  "metadata": {
    "name": "hourly",
    "namespace": "default",
    "labels": null,
    "annotations": null
  },
  "spec": {
    "action": "allow",
    "expressions": [
      "event.check.interval == 60",
      "event.check.occurrences == 1 || event.check.occurrences % 60 == 0"
    ],
    "runtime_assets": []
  }
}
```
### Create hourly filter
```
sensuctl create -f hourly.json
```
### List all filters
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl filter list
      Name        Action                               Expressions                               
 ─────────────── ──────── ──────────────────────────────────────────────────────────────────────                             
  hourly          allow    (event.check.interval == 60) && (event.check.occurrences == 1 || event.check.occurrences % 60 == 0)
```
### Add this filter to a handler
```
root@ip-172-31-38-43:/home/ubuntu# sensuctl handler update slack
? Environment variables: [? for help] (SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T0XXXX/B0XXXXX/XXXXXX) Environment variables: SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T0XXXX/B0XXXXX/XXXXXX
? Filters: hourly,is_incident,not_silenced
? Mutator: 
? Timeout: 0
? Type: pipe
? Runtime Assets: sensu-slack-handler
? Command: sensu-slack-handler --channel '#sensu-go'
Updated
```
Here we updated our custom filter hourly which will send notification to slack at the occurence of an event for the first time and once every hour. The filter is_incident and not_silenced are sensu in-built filters which will send notification to slack if there is an incident (i.e if check fails) and if the check is not silenced.

Combining these 3 filters we can reduce heavy noise on alerts triggered from sensu go and concentrate only on important alerts.

There are so many combinations you can use to create filters based on your requirement.
