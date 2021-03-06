# Container Runtime Interface (CRI) CLI

crictl provides a CLI for CRI-compatible container runtimes. This allows the CRI runtime developers to debug of their runtime without needing to set up Kubernetes components.

crictl is currently Alpha and still under quick iterations. We encourage the CRI developers to report bugs or help extend the coverage by adding more functionalities.

## Install

The CRI CLI can be installed easily via `go get` command:

```sh
go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
```

Then `crictl` binary can be found in `$GOPATH/bin`.

*Note: ensure GO is installed and GOPATH is set before installing crictl.*

## Usage

```sh
crictl SUBCOMMAND [FLAGS]
```

Subcommands includes:

- `info`: Get version information of runtime
- `sandbox`, `sb`: Manage lifecycle of podsandbox
- `container`, `ctr`: Manage lifecycle of container
- `status`: Get the status information of runtime
- `attach`: Attach a running container
- `image`: Manage images
- `exec`: Exec(exec, syncexec) a command in a running container
- `portforward`: Forword ports(localport:remoteport) from a sandbox
- `help`, `h`: Shows a list of commands or help for one command

crictl connects to `/var/run/dockershim.sock` by default. For other runtimes, the endpoint can be set in three ways:

- By setting flags `--runtime-endpoint` and `--image-endpoint`
- By setting environment variables `CRI_RUNTIME_ENDPOINT` and `CRI_IMAGE_ENDPOINT`
- By setting the endpoint in the config file `--config-file=/etc/crictl.yaml`

```
# cat /etc/crictl.yaml
runtime-endpoint: /var/run/dockershim.sock
image-endpoint: /var/run/dockershim.sock
timeout: 10
debug: true
```

## Additional options

- `--runtime-endpoint`: CRI server runtime endpoint (default: "/var/run/dockershim.sock").The default server is dockershim. If we want to debug other CRI server such as frakti, we can add flag `--runtime-endpoint=/var/run/frakti.sock`
- `--image-endpoint`: CRI server image endpoint, default same as runtime endpoint.
- `--timeout`: Timeout of connecting to server (default: 10s)
- `--debug`: Enable debug output
- `--help`, `-h`: show help
- `--version`, `-v`: print the version information of crictl
- `--config-file`: Config file in yaml format. Overrided by flags or environment variables.

## Examples

### Run sandbox with config file

```
# cat sandbox-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "linux": {
    }
}
# crictl sandbox run sandbox-config.json
9b542bfe8f93eb2d726d0f7b619f253c18858006aa53023e392e138b0be6301c
```

### Create container in a sandbox with config file

```
# cat sandbox-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "linux": {
    }
}
# cat container-config.json
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "linux": {
  }
}
# crictl container create 9b542bfe8f93eb2d726d0f7b619f253c18858006aa53023e392e138b0be6301c container-config.json sandbox-config.json
bf642f55ecf54345354a86a42c08fb0d66e55e90c855973495f31e991c2bf725
```

### Start container

```
# crictl container start bf642f55ecf54345354a86a42c08fb0d66e55e90c855973495f31e991c2bf725
bf642f55ecf54345354a86a42c08fb0d66e55e90c855973495f31e991c2bf725
```

### Exec a command in container

```
# crictl exec -i -t bf642f55ecf54345354a86a42c08fb0d66e55e90c855973495f31e991c2bf725 sh
```
