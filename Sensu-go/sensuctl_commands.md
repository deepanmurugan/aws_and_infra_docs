# Sensuctl commonly used commands

### Create resource from file
```
sensuctl create --file filename.json
```

### Sensuctl List commands
```
sensuctl [entity|asset|event|check|filter|handler|mutator|role|user...] list
```

### Sensuctl Update commands
```
sensuctl [check|handler|filter|event..] update [check_name|handler_name|filter_name|event_name]
```

### Sensuctl Info commands
```
sensuctl [check|handler|filter|event..] info [check_name|handler_name|filter_name|event_name] --format [json|yaml|wrapped-json...]
```
