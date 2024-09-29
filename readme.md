Shards Driver
=============

Kubernetes CSI driver assembling configuration files out of the shard resources.


# Example

```yaml
---
apiVersion: shards.driver/dev
kind: ConfigShard
metadata:
  name: test-shard-1
  labels:
    app: test
data:
  foo:
  - test1
  - test2
  bar:
    bar1: test1
---
apiVersion: shards.driver/dev
kind: ConfigShard
metadata:
  name: test-shard-2
  labels:
    app: test
data:
  foo:
  - test3
  - test4
  bar:
    bar2: test3
---
apiVersion: shards.driver/dev
kind: ConfigShard
metadata:
  name: test-shard-3
  labels:
    app: test
data:
  foobar: 123
```

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: busybox
    name: test
    command: ["tail"]
    args: ["-f", "/dev/null"]
    volumeMounts:
    - mountPath: /env.php
      name: php
    - mountPath: /env.json
      name: json
  volumes:
  - name: php
    csi:
      driver: shards.driver
      volumeAttributes:
        selector:
          app: test
        mode: php
  - name: json
    csi:
      driver: shards.driver
      volumeAttributes:
        selector:
          app: test
        mode: json
```

```
> kubectl exec test -- cat /env.php
<?php
return [
  "fobar" => 123,
  "bar" => [
    "bar1" => "test1",
    "bar2" => "test3",
  ],
  "foo" => [
    "test1",
    "test2",
    "test3",
    "test4",
  ],
];
> kubectl exec test -- cat /env.json
----
{
  "fobar": 123,
  "bar": {
    "bar1": "test1",
    "bar2": "test3"
  },
  "foo": [
    "test1",
    "test2",
    "test3",
    "test4"
  ]
}
```
