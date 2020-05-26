# Standalone checks in Sensu Go

Sensu Go has removed the standalone checks which were available in sensu core versions. To achieve the standalone logic you have to use subscriptions to achieve it.

Checks have a defined set of subscriptions: transport topics to which the Sensu backend publishes check requests. Sensu entities become subscribers to these topics (called subscriptions) via their individual subscriptions attribute. Subscriptions typically correspond to a specific role or responsibility (for example. a webserver or database).

Subscriptions also allow you to configure check requests for an entire group or subgroup of systems rather than requiring a traditional one-to-one mapping.

Read more about subscriptions [here](https://docs.sensu.io/sensu-go/latest/reference/checks/#subscriptions)

# Role Based Access Control

Sensu role-based access control (RBAC) helps different teams and projects share a Sensu instance. RBAC allows you to manage user access and resources based on namespaces, groups, roles, and bindings.

Read more about roles [here](https://docs.sensu.io/sensu-go/latest/reference/rbac/#roles-and-cluster-roles)
