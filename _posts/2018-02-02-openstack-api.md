---
layout: post
title:  "部署opsntack"
author: "Hill Zhang"
---
##　概念
1. User: user 是一类人或者能够通过keystone访问一些事情,user通过证书(username&password)或者api keys来访问服务
2. Project: Nova 创建的一个project, 一些资源的聚合,包括nova创建的虚拟机,neuton创建的网络,cinder创建的存储等, users 默认会被绑定到某个prokect
3. Role: 代表用户访问资源的权限，包括 admin,users，也可以根据自己的需求创建权限，不同的权限会被限制访问资源,　例如 user 被绑定到某个或某些project的同时会获取user操作该project的权限。
4. service: 服务类型 nova neuton glance
5. endpoint:　访问服务的入口，用户在通过keyston获取token之后或获取到服务的endpoint列表,此时用户可以通过endpoint来访问服务

##keystone(v3)

###　Tokens

1. 获取 unscoped　token
> 首先，用户需要知道能够访问哪些project，此时需要先想keystone获取　unscoped token, 这个token能够深入查询keystone service确定你能访问哪些prokect

```
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "Default"
                    },
                    "name": "demo",
                    "password": "123456"
                }
            }
        }
}
```
	
    待续

