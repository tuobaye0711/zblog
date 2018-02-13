---
title: 基于RBAC的分权分域用户权限系统数据库设计
date: 2018-02-12 16:02:24
tags: [mysql,技术杂谈]
description: 最近要开发一个新的系统，系统里面可以有许多不同的社区，每个社区下面有不同的项目。项目和项目、社区和社区之间都是相互独立的。我所做的工作是设计一套权限管理系统，这套系统允许不同的用户在不同的社区或者项目之中有不同的权限，可以保证用户无法做出越权的操作。
picture: 分权分域数据库设计.png
---

## 引子

***

最近要开发一个新的系统，系统里面可以有许多不同的社区，每个社区下面有不同的项目。项目和项目、社区和社区之间都是相互独立的。我所做的工作是设计一套权限管理系统，这套系统允许不同的用户在不同的社区或者项目之中有不同的权限，可以保证用户无法做出越权的操作。

经过了一番努力，设计出了一套基于RBAC的分权分域的用户权限系统。

首先介绍一下什么是RBAC:

> 以角色为基础的访问控制（英语：Role-based access control, RBAC），是资讯安全领域中，一种较新且广为使用的访问控制机制，其不同于强制访问控制以及自由选定访问控制直接赋予使用者权限，而是将权限赋予角色。1996年，莱威·桑度（Ravi Sandhu）等人在前人的理论基础上，提出以角色为基础的访问控制模型，故该模型又被称为RBAC96。之后，美国国家标准局重新定义了以角色为基础的访问控制模型，并将之纳为一种标准，称之为NIST RBAC。
> 以角色为基础的访问控制模型是一套较强制访问控制以及自由选定访问控制更为中性且更具灵活性的访问控制技术。

在一个组织中，会因为不同的作业功能产生不同的角色，执行某项操作的权限会被赋予特定的角色。组织成员或者工作人员（抑或其它系统用户）则被赋予不同的角色，这些用户通过被赋予角色来取得执行某项计算机系统功能的权限。

- S = 主体 = 一名使用者或自动代理人
- R = 角色 = 被定义为一个授权等级的工作职位或职称
- P = 权限 = 一种存取资源的方式
- SE = 会期 = S，R或P之间的映射关系
- SA = 主体指派
- PA = 权限指派
- RH = 角色阶层。能被表示为：≥（x ≥ y 代表 x 继承 y 的权限）
- 一个主体可对应多个角色。
- 一个角色可对应多个主体。
- 一个角色可拥有多个权限。
- 一种权限可被分配给许多个角色。
- 一个角色可以有专属于自己的权限。

## 数据库结构设计

***

项目使用的mysql数据库，设计表格的大体结构如下：

![数据库结构设计](分权分域数据库设计.png)

蓝底模块为**基本表**，黄底模块为**关系表**。

基本表有

- t_user
- t_role
- t_permission
- t_operation

关系表有

- t_user_role_community
- t_user_role_project
- t_role_permission
- t_permission_operation

权限判断的主要思路是根据用户传过来的社区ID/项目ID+用户ID+发起的请求路径，通过多表联查，来确定用户在对应的社区/项目内所对应的角色是否具有权限执行响应请求的操作。

由于在域方面，有社区和项目两个尺度，其中社区包含项目。但是由于一些其他原因，在社区表中无法表示出与项目的从属关系，只能把t_user_role表复用，分身成两张表：t_user_role_community和t_user_role_project。

由于我们的鉴权系统整个流程都是一对多的关系，即一个用户可能对应多个角色，一个角色可能对应多种权限，一种权限又能对应多种操作。因此，一个用户能对应N多种操作，同时，由于中间的多重映射关系，一个用户对某个操作可能会对应多次。

我们采用多表联查的方式，输入项为用户+操作（操作具体表示为request的method+url），使用如下SQL语句进行查询，只要查询结果不为空，即证明用户有相应的操作权限：

{% codeblock lang:sql %}
SELECT
    *
FROM
    t_user_role_community
INNER JOIN t_role_permission
INNER JOIN t_permission_operation ON t_user_role_community.strRole = t_role_permission.strRole
AND t_role_permission.nPermission = t_permission_operation.nPermission
WHERE
    nUser = 10025
AND nCommunity = 1
AND strMethod = 'get'
AND strUrl = '/api/t_service'
{% endcodeblock %}

这样做实际上只查询了社区线的权限，此外还有项目线的权限以及非社区非项目线的权限（例如个人中心、系统管理之类的，我们把这部分的域直接设置为nCommunity=0，即项目编号为0），想要覆盖所有的域，我们相当于要同时多表联查三次，只要其中任意一次能够查到数据，则表明用户有执行该操作的权限。我们可以用UNION语句来实现：

{% codeblock lang:sql %}
(SELECT
    *
FROM
    t_user_role_community
INNER JOIN t_role_permission
INNER JOIN t_permission_operation ON t_user_role_community.strRole = t_role_permission.strRole
AND t_role_permission.nPermission = t_permission_operation.nPermission
WHERE
    nUser = 10025
AND (nCommunity = 0 OR nCommunity = 0)
AND strMethod = 'get'
AND strUrl = '/api/t_service')
UNION
(SELECT
    *
FROM
    t_user_role_project
INNER JOIN t_role_permission
INNER JOIN t_permission_operation ON t_user_role_project.strRole = t_role_permission.strRole
AND t_role_permission.nPermission = t_permission_operation.nPermission
WHERE
    nUser = 10025
AND nProject = 0
AND strMethod = 'get'
AND strUrl = '/api/t_service')
{% endcodeblock %}

好啦，这就实现了一个相对完整的分权分域的系统啦~

## 结语

***

整个鉴权过程是放在后台以中间件的形式存在的。所有发送的请求都要经过鉴权中间件才能继续往后走下去。

因此，除了在数据库层面进行设计以外，还要在后台代码进行一些必要性的优化。比如说有一类操作的对应权限是通用权限，这类接口占所有接口的一半以上。因此在鉴权中间件我会先判断一步是否为通用权限，若是则跳过接下来的鉴权，直接判过；不是的话再按照既定流程继续鉴权。

也可以在鉴权层增加日志打印，把所有经过鉴权的请求记录在案，以便日后追踪。

写到这里突然发现，我这个前端，越做越往后了...