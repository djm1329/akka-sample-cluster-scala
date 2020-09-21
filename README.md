This repo is cloned from [akka-sample-cluster-scala](https://github.com/akka/akka-samples/tree/2.6/akka-sample-cluster-scala).

## A Simple Cluster Example Using IPv6

The code has been modified to use the IPv6 local loopback address (::1) instead of the IPv4 loopback address (127.0.0.1) to start the cluster. Akka Management has also been added, and started for the cluster.

Start a cluster, for example, using

```
sbt "runMain sample.cluster.simple.App 25251"
sbt "runMain sample.cluster.simple.App 25252"
```

in different terminals. By default Akka Management for each node will be started on <port + 10>.

Querying Akka Management for cluster members using

```
curl http://\[::1\]:25261/cluster/members | jq
```

yields

```
{
  "leader": "akka://ClusterSystem@[::1]:25251",
  "members": [
    {
      "node": "akka://ClusterSystem@[::1]:25251",
      "nodeUid": "-5574881386064603519",
      "roles": [
        "dc-default"
      ],
      "status": "Up"
    },
    {
      "node": "akka://ClusterSystem@[::1]:25252",
      "nodeUid": "-7069976705632475788",
      "roles": [
        "dc-default"
      ],
      "status": "Up"
    }
  ],
  "oldest": "akka://ClusterSystem@[::1]:25251",
  "oldestPerRole": {
    "dc-default": "akka://ClusterSystem@[::1]:25251"
  },
  "selfNode": "akka://ClusterSystem@[::1]:25251",
  "unreachable": []
}
```

However, trying to embed an IPv6 member address in Akka Management API calls that require an address does not appear to work:

```
curl http://\[::1\]:25261/cluster/members/ClusterSystem@\[::1\]:25251
```

gives

```
Illegal request-target: Invalid input '[', expected pchar, '/', '?' or 'EOI' (line 1, column 32)
```

The corresponding log from the Actor System at [::1]:25251 (corresponding to the Akka Management running at 25261) is 

```
[WARN] [akka.actor.ActorSystemImpl] [] [ClusterSystem-akka.actor.default-dispatcher-5] - Illegal request, responding with status '400 Bad Request': Illegal request-target: Invalid input '[', expected pchar, '/', '?' or 'EOI' (line 1, column 32): /cluster/members/ClusterSystem@[::1]:25251
```

The other API calls requiring embedded member addresses, such as downing a member, also exhibit the same problem. For example:

```
curl -XPUT -F 'operation=Down' http://\[::1\]:25261/cluster/members/ClusterSystem@\[::1\]:25252
```

yields

```
Illegal request-target: Invalid input '[', expected pchar, '/', '?' or 'EOI' (line 1, column 32)
```