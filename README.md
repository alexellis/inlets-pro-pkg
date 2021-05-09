# inlets-pro is a Cloud Native Tunnel for HTTP and TCP traffic

<img src="docs/images/inlets-pro-sm.png" width="150px">

# Overview

You can use inlets-pro to tunnel out any HTTP or TCP traffic from an internal network to another network. This could be green-to-green, or green-to-red, i.e. from internal/private to the Internet. It differs from the open source version in that it works at the L4 or L7 level of the TCP stack and has automatic TLS (auto-tls) encryption built-in.

Given the split control- and data-plane, you can also punch out endpoints into a remote cluster, which are kept private from the Internet, for instance when you need Command & Control, or orchestration of on-premises services, from a central cloud cluster.

## Features

inlets-pro forwards TCP traffic over an encrypted websocket secured with TLS.

* Support for any TCP protocol
* Pass-through for L4 proxy
* Reverse proxy and tunnel for L7 proxy
* Automatic Let's Encrypt when used as an L7 proxy
* Automatic TLS encryption for tunnel and control-port
* Automatic port-detection, announced by client

Deployment options:

* single static binary is available for MacOS, Windows, and Linux on armhf and ARM64
* `systemd` support with automatic restarts
* Native `docker` image available
* Kubernetes integration via `inlets-operator` or YAML

## License & Pricing

inlets-pro is a L4 and L7 TCP tunnel, service proxy, and load-balancer product distributed under a commercial license.

In order to use inlets-pro, you must accept the [End User License Agreement - EULA](EULA.md). The server component runs without a license key, but the client requires a valid license.

You can purchase a license for personal or business use on the [inlets website](https://inlets.dev/)

* [Purchase or start a free 14-day trial](https://inlets.dev)

## Reference architecture

inlets-pro can be used to provide a Public VirtualIP to private, edge and on-premises services and Kubernetes clusters. Once you have set up one or more VMs or cloud hosts on public cloud, you can utilize their IP addresses with inlets-pro.

You can get incoming networking (ingress) to any:

* gRPC services with or without TLS
* Access unsecured private services like MySQL, but with TLS link-encryption
* Command & control of Point of Sale / IoT devices
* SSH access to home-lab or Raspberry Pi
* TCP services running on Linux, Windows or MacOS
* The API of your Kubernetes cluster
* A VM or Docker container

For example, rather than terminating TLS at the edge of the tunnel, inlets-pro can forward the TLS traffic on port `443` directly to your host, where you can run a reverse proxy inside your network. At any time you can disconnect and reconnect the tunnel or even delete the remote VM without loosing your TLS certificate since it's stored locally.

See also: [reference architecture diagrams](/docs/reference.md)

## Get started

You can follow one of the tutorials above, or use inlets PRO in three different ways:

* As a stand-alone binary which you can manage manually or automate
* Through [inletsctl](https://github.com/inlets/inletsctl) which creates an exit server with `inlets-pro server` running with systemd in one of the cloud / IaaS platforms such as AWS EC2 or DigitalOcean
* Through [inlets-operator](https://github.com/inlets/inlets-operator) - the operator runs on Kubernetes and creates an exit server running `inlets-pro server` and a Pod in your cluster running `inlets-pro client`. The lifecycle of the client and server and exit-node are all automated.

### Tutorials and examples

* [News and use-cases on the blog](https://inlets.dev/blog)
* [Reference documentation](https://docs.inlets.dev)
* [inlets PRO CLI reference guide](docs/cli-reference.md)

### Get the binary

Both the client and server are contained within the same binary.

It is recommended that you use [inletsctl](https://github.com/inlets/inletsctl), or inlets-operator to access inlets-pro, but you can also work directly with its binary or Docker image.

The inlets-pro binary can be obtained as a stand-alone executable, or via a Docker image.

* As a binary:

    ```sh
    curl -SLsf https://github.com/inlets/inlets-pro/releases/download/0.8.1/inlets-pro > inlets-pro
    chmod +x ./inlets-pro
    ```

    Or fetch via `inletsctl download --pro`

    Or find a binary for [a different architecture on the releases page](https://github.com/inlets/inlets-pro/releases)

    See also [CLI reference guide](docs/cli-reference.md)

* Docker image

    A docker image is published at `ghcr.io/inlets/inlets-pro:0.8.1`
    
    See the image on [GitHub Container Registry](https://github.com/orgs/inlets/packages/container/package/inlets-pro)

### Kubernetes

* Kubernetes LoadBalancer integration

    See also: [inlets-operator](https://github.com/inlets/inlets-operator)

* Kubernetes Helm charts

    Run ad-hoc clients and servers on your Kubernetes clusters

    See [chart](chart) for the inlets-pro TCP client and server

* Sample Kubernetes YAML files

    A [client](artifacts/client.yaml) and [server](artifacts/server.yaml) YAML file are also available as samples

## Get in touch

Got questions? Send us an email to [contact@openfaas.com](mailto:contact@openfaas.com).
