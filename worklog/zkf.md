# 互联网模板 和 一体化模板

```
淮安核心域
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
## 互联网模板

原模板位置：
172.19.77.4 /opt/software/nginx_update/tmpl/internet/huitui/nginx.tmpl

1. 创建nginx的互联网模板
cd /opt/software/nginx_update/tmpl/internet/huitui/
kubectl delete cm -n kube-system nginx-internet-tmpl 
kubectl create cm -n kube-system nginx-internet-tmpl --from-file=nginx.tmpl
2. 进入备份ds和config文件
cd /opt/software/nginx_update/int/hlwzx-restapi-1/
kubectl replace -f cm-bak.yaml 
kubectl replace -f ds-bak.yaml 
3. 删除pod
kubectl delete pod -n kube-system ingress-controller-hlwzx-restapi-1-xxx

## 一体化模板

原模板位置：

172.19.77.4 /opt/software/nginx_update/tmpl/yth/huitui/nginx.tmpl 

1. 创建nginx的一体化模板
cd /opt/software/nginx_update/tmpl/yth/huitui/
kubectl delete cm -n kube-system nginx-tmpl 
kubectl create cm -n kube-system nginx-tmpl --from-file=nginx.tmpl
2. 进入备份ds和config文件
cd /opt/software/nginx_update/yth/itxtb-restapi-1/
kubectl replace -f cm-bak.yaml 
kubectl replace -f ds-bak.yaml 
3. 删除pod
kubectl delete pod -n kube-system ingress-controller-itxtb-restapi-1-xxx

> 注意:重创nginx.tmpl后需要打标签.

## 修改密码
```
skip=14

REAL_IP=`/sbin/ip a | /bin/grep -w inet | /bin/grep -w brd | grep 192.168|/bin/awk '{print $2}' | /bin/awk -F "/" '{print $1}'`

REAL_PASSWD=`tail -n +$skip "$0" | grep -w ${REAL_IP} | awk '{print $2}'`

#echo  "DEBUG:" $REAL_IP $REAL_PASSWD
echo "$REAL_PASSWD" | passwd --stdin root


exit 0

#Please repleace "IP PASSWD" as follow:
192.168.159.179	1q2w!Q@W
192.168.159.177	1q2w!Q@W
192.168.159.176	1q2w!Q@W
192.168.159.180	1q2w!Q@W
192.168.159.175	1q2w!Q@W
192.168.159.181	1q2w!Q@W
192.168.159.182	1q2w!Q@W
192.168.159.183	1q2w!Q@W
192.168.159.220	1q2w!Q@W
192.168.159.219	1q2w!Q@W
192.168.159.221	1q2w!Q@W
192.168.159.223	1q2w!Q@W
192.168.159.222	1q2w!Q@W
192.168.159.224	1q2w!Q@W
192.168.159.225	1q2w!Q@W
192.168.159.226	1q2w!Q@W
192.168.159.227	1q2w!Q@W
192.168.159.228	1q2w!Q@W
192.168.159.190	1q2w!Q@W
192.168.159.189	1q2w!Q@W
192.168.159.191	1q2w!Q@W
192.168.159.192	1q2w!Q@W
192.168.159.193	1q2w!Q@W
192.168.159.194	1q2w!Q@W
192.168.159.195	1q2w!Q@W
192.168.159.196	1q2w!Q@W
192.168.159.197	1q2w!Q@W
192.168.159.198	1q2w!Q@W
192.168.159.204	1q2w!Q@W
192.168.159.205	1q2w!Q@W
192.168.159.206	1q2w!Q@W
192.168.159.207	1q2w!Q@W
192.168.159.209	1q2w!Q@W
192.168.159.208	1q2w!Q@W
192.168.159.210	1q2w!Q@W
192.168.159.211	1q2w!Q@W
192.168.159.212	1q2w!Q@W
192.168.159.213	1q2w!Q@W
```

