## 描述
目前nginx版本已经升级到1.15 ，upstream模块交给lua来管理。

## 负载均衡对应的backends
>淮安核心域
```shell
default          8181
yth2-restapi-1   8182
hlwzx-web-1      8183
yth-web-1        8184
itxtb-restapi-1  8185
itxtb-web-1      8186
yth-nginx-a      8187
yth-nginx-hd     8188
yth-nginx-hd-x   8189
yth-restapi-1    8190
hlwzx-restapi-1  8191
internet         8192
wltx-restapi-1   8193
wltx-web-1       8194
yth-restapi-2    8195
yth-web-2        8196
yth2-web-1       8197
```

>洛阳核心域
```shell
nginx-ingress-controller 8181
yth2-restapi-1 8182
hlwzx-web-1 8183
yth-web-1 8184
itxtb-restapi-1 8185
itxtb-web-1 8186
yth-nginx-a 8187
yth-nginx-hd 8188
yth-nginx-hd-x 8189
yth-restapi-1 8190
hlwzx-restapi-1 8191
internet 8192
wltx-restapi-1 8193
wltx-web-1 8194
yth-restapi-2 8195
yth-web-2 8196
yth2-web-1 8197
night-nginx-hd 8198
n1 8199
intelligentize 8200
```
## 查询某一个负载均衡下对应的upstream
```shell
default          http://172.19.77.12:8181/backends
yth2-restapi-1   http://172.19.77.12:8182/backends
hlwzx-web-1      http://172.19.77.12:8183/backends
yth-web-1        http://172.19.77.58:8184/backends
itxtb-restapi-1  http://172.19.77.12:8185/backends
itxtb-web-1      http://172.19.77.12:8186/backends
yth-nginx-a      http://172.19.77.58:8187/backends
yth-nginx-hd     http://172.19.77.58:8188/backends
yth-nginx-hd-x   http://172.19.77.58:8189/backends
yth-restapi-1    http://172.19.77.58:8190/backends
hlwzx-restapi-1  http://172.19.77.12:8191/backends
internet         http://172.19.77.12:8192/backends
wltx-restapi-1   http://172.19.77.12:8193/backends
wltx-web-1       http://172.19.77.12:8194/backends
yth-restapi-2    http://172.19.77.58:8195/backends
yth-web-2        http://172.19.77.58:8196/backends
yth2-web-1       http://172.19.77.58:8197/backends
```
## upstream详细信息
```shell
{
  "name": "itxtbqt-hacoreprd-tpms-web-hua-prd-8080",  #命名空间  服务名  端口
  "service": {
    "metadata": {
      "creationTimestamp": null
    },
    "spec": {
      "ports": [
        {
          "name": "port-5da4c3bf-dd1e-4f55-a857-e6cfcbae9f97",
          "protocol": "TCP",
          "port": 8080,
          "targetPort": 8080
        }
      ],
      "selector": {
        "app": "tpms-web-hua-prd",
        "harmonycloud.cn/bluegreen": "tpms-web-hua-prd-1"
      },
      "clusterIP": "172.25.11.2",
      "type": "ClusterIP",
      "sessionAffinity": "None"
    },
    "status": {
      "loadBalancer": {}
    }
  },
  "port": 8080,
  "secureCACert": {
    "secret": "",
    "caFilename": "",
    "pemSha": ""
  },
  "sslPassthrough": false,
  "endpoints": [
    {
      "address": "172.24.178.168",
      "port": "8080"
    },
    {
      "address": "172.24.250.1",
      "port": "8080"
    }
  ],
  "sessionAffinityConfig": {
    "name": "",
    "cookieSessionAffinity": {
      "name": ""
    }
  },
  "upstreamHashByConfig": {
    "upstream-hash-by-subset-size": 3
  },
  "noServer": false,
  "trafficShapingPolicy": {
    "weight": 0,
    "header": "",
    "headerValue": "",
    "cookie": ""
  }
}
```

## 查询某一个服务下对应的upstream

```shell
cat lua.txt | jq '.[0]'   #查看第一个数组
cat lua.txt | jq '.[]|{name:.name}'   #查看所有的服务
{
  "name": "itframe-hadrpprd-drpweb-hua-prd-8080"
}
{
  "name": "itframe-huacoreprd-csfadmin-hua-prd-8080"
}
{
  "name": "itframe-huacoreprd-ngtask-hua-prd-b-8080"
}
{
  "name": "itframe-huacoreprd-wf-manager-hua-prd-8080"
}
{
  "name": "itxtbqt-hacoreprd-tpms-control-hua-prd-8080"
}
{
  "name": "itxtbqt-hacoreprd-tpms-web-hua-prd-8080"
}
{
  "name": "upstream-default-backend"
}

cat lua.txt | jq '.[]|{name:.name ,  endpoints:.endpoints}'   #查看所有的服务的ENDPOINT
{
  "name": "itxtbqt-hacoreprd-tpms-web-hua-prd-8080",
  "endpoints": [
    {
      "address": "172.24.178.168",
      "port": "8080"
    },
    {
      "address": "172.24.250.1",
      "port": "8080"
    }
  ]
}
{
  "name": "upstream-default-backend",
  "endpoints": [
    {
      "address": "172.24.197.160",
      "port": "8080"
    }
  ]
}
```