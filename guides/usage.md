# Usage

* [Connections (`K8s.Conn`)](./connections.html)
* [Operations (`K8s.Operation`)](./operations.html)
* [Discovery (`K8s.Discovery`)](./discovery.html)
* [Middleware (`K8s.Middleware`)](./middleware.html)
* [Authentication (`K8s.Conn.Auth`)](./authentication.html)
* [Observability](./observability.html)
* [Testing](./testing.html)
* [Advanced Topics](./advanced.html) - CRDs, Multiple Clusters, and Subresource Requests

## tl;dr Examples

### Creating a deployment

```elixir
{:ok, conn} = K8s.Conn.from_file("path/to/kubeconfig.yaml")

opts = [namespace: "default", name: "nginx", image: "nginx:nginx:1.7.9"]
{:ok, resource} = K8s.Resource.from_file("priv/deployment.yaml", opts)

operation = K8s.Client.create(resource)
{:ok, deployment} = K8s.Client.run(conn, operation)
```

### Listing deployments

In a namespace:

```elixir
{:ok, conn} = K8s.Conn.from_file("path/to/kubeconfig.yaml")

operation = K8s.Client.list("apps/v1", "Deployment", namespace: "prod")
{:ok, deployments} = K8s.Client.run(conn, operation)
```

Across all namespaces:

```elixir
{:ok, conn} = K8s.Conn.from_file("path/to/kubeconfig.yaml")

operation = K8s.Client.list("apps/v1", "Deployment", namespace: :all)
{:ok, deployments} = K8s.Client.run(conn, operation)
```

### Getting a deployment

```elixir
{:ok, conn} = K8s.Conn.from_file("path/to/kubeconfig.yaml")

operation = K8s.Client.get("apps/v1", :deployment, [namespace: "default", name: "nginx-deployment"])
{:ok, deployment} = K8s.Client.run(conn, operation)
```

### Executing a command

If your Pod has only one container, then you do not have to specify which container to run the command in.

```elixir
  {:ok, conn} = K8s.Conn.from_file("~/.kube/config")

  connect = K8s.Client.connect("v1", "pods/exec", [namespace: "default", name: "nginx-8f458dc5b-zwmkb"])

  op = K8s.Operation.put_query_param(connect, [command: ["/bin/sh", "-c", "nginx -t"], stdin: true, stdout: true, stderr: true, tty: true])

  K8s.Client.run(conn, op, stream_to: self())

  # wait for the response from the pod
  receive do
    {:ok, message} -> #do something with the messages. There can be a lot of output.
    {:exit, {:remote, 1000, ""}} -> # The websocket closed because of normal reasons.
    error -> # Something unexpected happened.
  after
    60_0000 -> Process.exit(pid, :kill) # we probably dont want to let this run forever as this can leave orphaned processes.
  end
```

Same as above, but you explicitly set the container name you want to run a command in.

```elixir
  {:ok, conn} = K8s.Conn.from_file("~/.kube/config")
  connect = K8s.Client.connect("v1", "pods/exec", [namespace: "default", name: "nginx-8f458dc5b-zwmkb"])

  op = K8.Operation.put_query_param(connect, [command: ["/bin/sh", "-c", "nginx -t"], container: "fluentd", stdin: true, stdout: true, stderr: true, tty: true])

  K8s.Client.run(conn, op, stream_to: self())

  receive do
    {:ok, message} -> # Do something with the messages. There can be a lot of output.
    {:exit, {:remote, 1000, ""}} -> # The websocket closed because of normal reasons.
    error -> # Something unexpected happened.
  after
    60_0000 -> Process.exit(pid, :kill) # we probably dont want to let this run forever as this can leave orphaned processes.
  end
```
