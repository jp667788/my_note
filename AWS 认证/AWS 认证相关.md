
# AWS Regions 区域

- 这些 Regions 地区遍布世界各地
- 控制台中可以看到对应的 Regions 列表
- 它是一个数据中心集群

> 如何选择 AWS 区域？
> 影响选择的因素：
> - 合规  
> - 延迟：尽可能近的部署
> - 所选择的区域有可用的服务
> - 价格


# AWS Availability Zones 可用区域

- 可用区域之间容灾
- 可用区域间通过高带宽、超低延迟的网络连接在一起，形成一个区域 （Regions）

# AWS Points of Presence (Edge Locations) 接入点（边缘位置）


- AWS在40个国家/地区的90个城市拥有400多个网点｡


# IAM ：Users and Group 用户和组

- IAM代表身份 和访问管理，这是一项全球服务
- 根用户默认创建，使用它只是为了设置帐户
- 用户代表一个用户组织内的成员
- 组只包含了用户，不能包含其他组
- 一个用户可以同时在不同组

# IAM：Permissions （权限）

- 用户和组的权限可以通过JSON进行配置
- 策略决定了用户的权限范围
- 遵守最小特权原则的原则

# IAM Polices (策略)

- 一个用户可能不属于任何组，可以创建一个内联策略


## 策略结构

- Version 版本号：策略语言版本
- id 标识策略ID
- Statement 可以是多个
	- SID：语句的标识符，可选的
	- Effect：策略定位的效果，即该语句是否允许或拒绝访问特定API
	- Principal: 原则包括将应用此策略的帐户､ 用户或角色
	- Action: 操作是将根据效果拒绝或允许的API调用的列表
	- Resource: 资源是操作将应用到的资源的列表｡

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/2023-05-08-084504.png)


# Multi Factor Authentication - MFA 多因素身份认证

- 用户可以访问您的帐户，他们可以执行许多操作，特别是如果他们是管理员，他们可以更改配置､ 删除资源和其他操作｡
- 除了密码外，使用 MFA 设备保护帐户
- MFA使用的是你知道的密码和你拥有的安全设备的组合，这两样东西结合在一起，比仅仅一个密码有更大的安全性
- MFA 优势在于，即使丢失密码，也能够抱回帐户不被损害

# 如何访问 AWS

- AWS 控制台 :（密码 + MFA）
- AWS CLI 命令行界面: access keys
- AWS SDK: access keys

##  acccess key

- acccess key 通过控制台生成
- 用户自己管理自己的 key
- acccess key 相当于密码，需要保密


# IAM Roles 