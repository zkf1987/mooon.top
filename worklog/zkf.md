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


