# inlets-pro CLI reference

There are two components to inlets-pro, the server and the client.

## Working with MacOS, Linux, and Windows

The examples given in the documentation are valid for all three operating systems and use bash as a syntax.

Windows users can use either Windows Subsystem for Linux (WSL) or [Git bash](https://git-scm.com/downloads), this is the simplest way to make all commands compatible.

Both the client and server component can run as:

* A process on MacOS, Linux, Windows on ARM or Intel architecture
* As a Docker container with docker, or Kubernetes as a Pod on ARM or Intel architecture

### Configure the inlets-pro client

The client component connects to an inlets server and then routes incoming requests to a private service. The client can run on the same host as your private service, or run on another host and act as gateway.

#### Configure the license

The license terms of inlets-pro require that both the inlets client and server have a valid license, only the client requires to have the license configured.

You can configure the license in one of two ways:

* From a file `--license-file`

    ```sh
    # Assume a file of `pro-license.txt` with the license key, no new lines or whitespace
    inlets-pro client \
    --license-file=pro-license.txt
    ```

* literal flag `--license`

    ```sh
    inlets-pro client \
    --license="LICENSE_KEY_VALUE"
    ```

* literal flag with environment variable

    ```sh
    export INLETS_LICENSE="LICENSE_KEY_VALUE"
    inlets-pro client \
    --license="$INLETS_LICENSE"
    ```

* literal flag with environment variable set in your bash profile

    You can also set the INLETS_LICENSE file for each terminal session by editing `$HOME/.bash_profile`

    Add a line for:

    ```sh
    export INLETS_LICENSE="LICENSE_KEY_VALUE"
    ```

#### Configure the routing

Each inlets-server and client pair acts as a router. You need to configure the client to tell it where to route incoming TCP requests and which port to use.

##### Remote TCP address `--remote-tcp`

* For a client running on your local computer or a VM

    Set `--remote-tcp` - set to `127.0.0.1` for the local machine

* For a client acting as a gateway, specify the hostname or IP address as seen by the client

    Set `--remote-tcp` - set to `192.168.0.1` if the host running your private service is `192.168.0.1` on the local network

* For a Kubernetes Pod

    Set `--remote-tcp` - set to the name of the destination Kubernetes service such as a ClusterIP `nginx.default`

##### TCP ports `--tcp-ports`

The client will advertise which TCP ports it requires the server to open, this is done via the `--tcp-ports` flag

* A single alternative HTTP port

    `--tcp-ports=8080`

* Nginx, or a HTTP service with TLS

    `--tcp-ports=80,443`

#### Connect to the remote host with `--connect`

inlets-pro uses a websocket for its control plane on port `8123` by default and adds automatic TLS. This is an optional feature.

* Automatic TLS with `auto tls`

    In this mode the client and server will negotiate TLS through the use of a generate Certificate Authority (CA) and encrypt all traffic automatically.

    This is the default option, connect with `wss://` and the IP of the remote machine

    `--connect wss://remote-machine:8123/connect`

    The control-port of 8123 is used for auto-tls.

* External TLS

    In this mode, you are providing your own TLS certificate or termination through a gateway, IngressController, reverse-proxy or some other kind of product.

    Turn auto-TLS off, and use port 443 (implicit) for the control-plane.

    `--connect wss://remote-machine/connect`

    You must also pass the `--auto-tls=false` flag

* No TLS or encryption

    This mode may be useful for testing, but is not recommended for confidential use.

    `--connect ws://remote-machine:8123/connect`

    Use port `8123` for the control-plane and `ws://` instead of `wss://`

#### Set the authentication token `--token`

The inlets-pro server requires a token for authentication to make sure that the client is genuine. It is recommended to combine the use of the token with auto-tls or external TLS.

You can create your own token, or generate one with bash:

```sh
export TOKEN="$(head -c 16 /dev/urandom |shasum|cut -d'-' -f1)"
echo $TOKEN
```

Now pass the token via `--token $TOKEN`.

### Configure the inlets-pro server

The inlets-pro server begins by opening a single TCP port for the control-plane, this is port `8123`, but you can customise it if required.

Additional ports are opened at runtime by the inlets-server for the data-plane. These ports must be advertised by the client via the `--tcp-ports` flag.

#### Start with auto-tls

Auto-TLS will create a Certificate Authority CA and start serving it via the control-plane port.

You can view it like this:

```sh
curl -k -i http://localhost:8123/.well-known/ca.crt
```

A token is also required which must be shared with the client ahead of time.

#### Set the `--common-name`

The `--common-name` is part of the auto-tls configuration and is used to configure the certificate-authority.

You can use the public IP address of the inlets-server here, or a DNS record.

* Public IP

    ```sh
    --common-name 35.1.25.103
    ```

* DNS A or CNAME record

    ```sh
    --common-name inlets-control-tunnel1.example.com
    ```

    In this example `inlets-control-tunnel1.example.com` will resolve to the public IP of `35.1.25.103`

#### Set the authentication token `--token`

The inlets-pro server requires a token for authentication to make sure that the client is genuine. It is recommended to combine the use of the token with auto-tls or external TLS.

You can create your own token, or generate one with bash:

```sh
export TOKEN="$(head -c 16 /dev/urandom |shasum|cut -d'-' -f1)"
echo $TOKEN
```

Now pass the token via `--token $TOKEN`.

## Working with Kubernetes

You can deploy an inlets-server in one of three ways:

* As a Service type LoadBalancer

    It will gain its own IP address, and you'll pay for one cloud load-balancer per tunnel. This is the easiest option, and has full encryption with auto-TLS and adds 15-20USD / per IP.

* As a Service type NodePort

    You will have to use high, non-standard TCP ports and may run into issues with manually managing the mapping of ports. This adds no cost to the Kubernetes cluster. You can also use auto-TLS for the control-plane.

* As an Ingress definition

    The Ingress definition is the most advanced option and works without auto-TLS. For each inlets-server you need to create a separate Kubernetes Ingress definition and domain name.

    Clients will connect to the domain name and your IngressController will be responsible for configuring TLS either via LetsEncrypt or your own certificate store.

### Pod / Service / Deployment definitions

You can use the sample artifact for the [client.yaml](artifacts/client.yaml) or [server.yaml](artifacts/server.yaml)

### Common issues

* You have a port permission issue for low ports such as 80

    This may be a permission issue with your cluster. Contact your administrator.

    Try adding each port to the Kubernetes container spec:

    ```yaml
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
    ```

    You may also need to run the pod as a root user by [editing the security context of the Pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

* The client cannot write the auto-TLS certificate to `/tmp/` due to a read-only filesystem

    Add a tmpfs mount or an empty-dir mount to the Pod Spec at `/tmp/`

    ```yaml
    volumes:
    - name: tmp-cert
    emptyDir: {}
    ```

    To the container spec:

    ```yaml
    volumeMounts:
    - mountPath: /tmp
        name: tmp-cert
    ```

