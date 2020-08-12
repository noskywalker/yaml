#自定义ipam，如使用calico，calico有自己的ipam插件，在/opt/cni/bin目录下
## 修改calico的config-map
```cfml
            "ipam": {
               "type": "calico-ipam",#启用calico-ipam
               "subnet": "usePodCidr"
             },
             "feature_control": {
                "ip_addrs_no_ipam": true # 启用(非必须)
             },
```
使用k8s注解来定义固定ip
```cfml
kind: Pod
metadata:
   name: spring-web-pod
   labels:
     name: spring-web-pod
   annotations:
    "cni.projectcalico.org/ipAddrs": "[\"172.16.1.189\"]"
```
#或者使用deployment来管理pod，指定一个较小范围的固定ip，使得多副本可以固定ip
```cfml
kind: Deployment
metadata:
  name: centos-deploy-ip-range
spec:
  selector:
    matchLabels:
      app: centos-svc
  replicas: 3
  template:
    metadata:
      labels:
        app: centos-svc
      annotations:
        "cni.projectcalico.org/ipv4pools": "[\"myself-pool\"]"
```
需要指定一个ippool:
```cfml
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: myself-pool
spec:
  blockSize: 29
  cidr: 172.17.1.0/29
  ipipMode: Always
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
```

参见calico官方文档：
https://docs.projectcalico.org/reference/cni-plugin/configuration#using-host-local-ipam