---
layout: post
title: "What's inside the Nginx Ingress Controller?"
excerpt: "This post explores the inner workings of the Nginx Ingress Controller and provides useful insights for application developers."
modified: 2024-03-15 12:05:18 +0200
categories: articles
tags: [kubernetes, nginx]
image:
  path: /images/2024-03-03-whats-inside-the-nginx-ingress-controller/cover.jpg
  thumbnail: /images/2024-03-03-whats-inside-the-nginx-ingress-controller/cover_thumb.jpg
  caption: "Photo by [Taylor Vick](https://unsplash.com/photos/cable-network-M5tzZtFCOfs)"
comments: true
share: true
published: true
aging: false
amazon_links: false
---

When working with applications in Kubernetes, there's a high chance that you've come across the [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx).
But what exactly makes it *Nginx*?
From the application developer's perspective, we usually don't think about the implementation of the Ingress Controller.
We use the Ingress API to define the virtual host and declare routes to services.
In this post, we'll explore the inner workings of the Nginx Ingress Controller and show you that it's essentially an Nginx reverse proxy running in a container.

## Where Can I Find Nginx?

To answer the question, let's take out our trusty old `kubectl` and pry open an Nginx Ingress Controller.

```bash
$ kubectl get pods -n ingress-nginx

NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-controller-7dcdbcff84-bf6sn   1/1     Running     0          40s
ingress-nginx-controller-7dcdbcff84-mvkkj   1/1     Running     0          2m12s
```

Let's exec into a pod and explore.

```bash
$ kubectl exec --stdin --tty -n ingress-nginx ingress-nginx-controller-7dcdbcff84-bf6sn -- /bin/bash
```
At `/etc/nginx` we can find a file called `nginx.conf`.
The file name should be familiar if you’ve ever set up an Nginx reverse proxy
This is the main configuration file for the Nginx web server.

The default configuration file is pretty dense.
Feel free to look inside but in the context of this post the details of the configuration aren't relevant.
The following is a rough outline.

```
http {
  
    # Server configuration
    server {
        listen 80;
        server_name localhost;
        location / {
        }
    }

    server {
        location / {
        }
    }
}
```

* The `http` block configures the HTTP server
* Each `server` block represents a virtual server that listens for HTTP requests on a specific IP address and port
* The `location` block is used to define how the server should handle different URLs within a given server block

In the grand scheme of things, the configuration's structure is fairly similar to the ingress resource rules where we define hosts and paths. 
Let's dig deeper and see what happens when we create a new ingress.

## Create a New Ingress

To better understand how the Nginx Ingress works, let’s create a new ingress.
The following configuration creates a Kubernetes deployment, service and an ingress for [`echo-server`](https://github.com/jmalloc/echo-server), a simple HTTP server that echoes the information about the incoming HTTP request back to the client.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo-server
  template:
    metadata:
      labels:
        app: echo-server
    spec:
      containers:
        - name: echo-server
          image: jmalloc/echo-server
          ports:
            - name: http-port
              containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: echo-service
spec:
  ports:
    - name: http-port
      port: 80
      targetPort: http-port
      protocol: TCP
  selector:
    app: echo-server
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: foo.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: echo-service
              port:
                name: http-port
        - path: /bar
          pathType: Prefix
          backend:
            service:
              name: echo-service
              port:
                name: http-port
```

Have a look at the ingress configuration.
Take note of the host `foo.com` and the two paths.

Assuming the entire configuration is in a file named `echo-server.yml`, let's create the Kubernetes resources using `kubectl`.
```
$ kubectl apply -f echo-server.yml
```

When we now look at the `nginx.conf` file, a new server block has appeared for `foo.com`.

```
## start server foo.com
server {
	server_name foo.com ;
```

The `foo.com` server block has three location blocks defined.

```
location /bar/ {
  ...

location = /bar {
  ...

location / {
  ...
```

Based on this observation, we can conclude that whenever the state of the objects in the Kubernetes cluster changes, the `nginx.conf` file is updated accordingly.

## Proxy Upstream

In the context of a Kubernetes ingress, Nginx acts as a reverse proxy.
It accepts HTTP connections and proxies them to upstream backends, which in our case are pods.
Let's look at the `nginx.conf` again and see where the upstream backends are defined.

In the `foo.com` server location blocks, we can see the following.

```
proxy_pass http://upstream_balancer;
```

And if we look at the upstream block, it's defined as follows.

```
upstream upstream_balancer {
	server 0.0.0.1:1234; # placeholder
	
	balancer_by_lua_block {
		tcp_udp_balancer.balance()
	}
}
```

Usually, when defining Nginx upstream servers manually, we list all the servers in the upstream block.
However, Nginx Ingress Controller doesn't do that.
If it did, the configuration would have to be reloaded every time a pod is added or removed.
Instead, it uses the [`lua-nginx-module`](https://github.com/openresty/lua-nginx-module) to update the upstream configuration.

> In a relatively big cluster with frequently deploying apps this feature [lua-nginx-module] saves significant number of Nginx reloads which can otherwise affect response latency, load balancing quality (after every reload Nginx resets the state of load balancing) and so on.
>
> <footer><a href="https://kubernetes.github.io/ingress-nginx/how-it-works/#avoiding-reloads-on-endpoints-changes">Avoiding reloads on Endpoints changes &mdash; Nginx Ingress Controller Documentation</a></footer>

The Nginx Ingress Controller has a binary called `dbg` in the root directory which we can use to list the current upstream backends.

```bash
$ /dbg backends all
```

The command will output a JSON array containing all currently configured backends.
In our case, the one we're interested in is the `echo-service`.
The following output is considerably truncated.
In reality, the array has more entries and the objects have more attributes that aren't significant in the context of this post.

```json
[
  {
    "name": "default-echo-service-http-port",
    "endpoints": [
      {
        "address": "10.244.1.60",
        "port": "8080"
      },
      {
        "address": "10.244.0.129",
        "port": "8080"
      },
      {
        "address": "10.244.0.69",
        "port": "8080"
      }
    ]
  }
]
```

The list of endpoints in the `dbg` output matches the `echo-service` k8s endpoints.

``` bash
$ kubectl get endpoints echo-service
NAME           ENDPOINTS                                             AGE
echo-service   10.244.0.129:8080,10.244.0.69:8080,10.244.1.60:8080   15m
```

Essentially, these are the IP addreses of the `echo-service` pods.

## How `nginx.conf` Gets Updated

In addition to the Nginx binary, the Nginx Ingress Controller container has a process called `nginx-ingress-controller`.

```bash
$ ps -a
PID   USER     TIME  COMMAND
    1 www-data  0:00 /usr/bin/dumb-init -- /nginx-ingress-controller --publish-service=ingress-nginx/ingress-nginx-controller --election-id=ingress-nginx-leader --controller-class=k8s.io/ingress-nginx --ingress-cla
    7 www-data  0:00 /nginx-ingress-controller --publish-service=ingress-nginx/ingress-nginx-controller --election-id=ingress-nginx-leader --controller-class=k8s.io/ingress-nginx --ingress-class=nginx --configmap=i
   19 www-data  0:00 nginx: master process /usr/bin/nginx -c /etc/nginx/nginx.conf
   25 www-data  0:00 nginx: worker process
   26 www-data  0:00 nginx: cache manager process
```

It's a [Kubernetes controller](https://kubernetes.io/docs/concepts/architecture/controller/) that talks to the Kubernetes API server and [watches for changes in the state of the cluster](https://kubernetes.github.io/ingress-nginx/how-it-works/#nginx-model).
If Kubernetes resources such as Ingresses, Services, Endpoints, Secrets or Configmaps are updated, `nginx-ingress-controller` will update the Nginx configuration file and reload the configuration if needed.

## Summary

In the book [Fundamentals of Software Architecture](https://www.oreilly.com/library/view/fundamentals-of-software/9781663728357/), the authors defined the first law of software architecture: *“Everything in software architecture is a trade-off”*.
To be able to assess the trade-offs properly, it's good to have a deeper understanding of the components we're working with and not treat them as black boxes.
Knowing how things work under the hood is a superpower.

When it comes to the Nginx Ingress Controller, we've learned that it's essentially an Nginx reverse proxy running in a container with some extra functionality that keeps the configuration in sync with the state of the Kubernetes cluster.