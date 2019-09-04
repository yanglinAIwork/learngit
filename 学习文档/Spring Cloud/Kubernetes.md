## Kubernetes

### 一、Kubernetes授权控制原则

​		Kubernetes在**授予访问权限时采用最小授权原则**。例如：如果某个 Pod 使用了特定的 `serviceAccount`，那么该 Pod 被限定为只能拥有指定的权限，只能访问特定的资源。

​		Kuberneters从1.6开始支持基于角色的访问控制机制（Role-Based Access，<a href="#一、RBAC">RBAC</a>），集群管理员可以对用户或服务账号的角色进行更精确的资源访问控制。

​		



---

## 附录

### 一、RBAC

#### 1、什么是RBAC

​		`RBAC`是基于角色的访问控制（`Role-Based Access Control` ），在`RBAC`中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。这就极大地简化了权限的管理。这样管理都是层级相互依赖的，权限赋予给角色，而把角色又赋予用户，这样的权限设计很清楚，管理起来很方便。

​		`RBAC`原理：`RBAC` 授权策略会创建一系列的 `Role` 和 `ClusterRole` 来绑定相应的资源实体（serviceAccount 或 group），以此来限制其对集群的操作。每一个 Role 都基于 Create, Read, Update, Delete（CRUD）模型来构建，并使用“动词”来应用相应的权限。例如，动词 `get` 表示能够获取特定资源的详细信息。如果你想获取对 `Secrets` 的访问权限，可以创建如下的 ClusterRole：

![Kubernetes](F:\GitDepository\学习文档\Spring Cloud\baseImages\Kuberneters-1.jpg)

---

#### 2、RBAC四大结构

​		RBAC 又分为`RBAC0、RBAC1、RBAC2、RBAC3`

- **RBAC0：是RBAC的核心思想。**
- **RBAC1：是把RBAC的角色分层模型。**
- **RBAC2：增加了RBAC的约束模型。**
- **RBAC3：其实是RBAC2 + RBAC1。**

---

##### a、RBAC0是RBAC的核心

​	RBAC0是RBAC的核心部分，主要有四部分组成。

	1. 用户（User）
 	2. 角色（Role）
 	3. 许可（Permisson）
 	4. 会话（Session）

![1566811580876](F:\GitDepository\学习文档\Spring Cloud\baseImages\RBAC0.png)

---

##### b、RBAC1，基于角色的分层模型

​	RBAC1是对RBAC0的扩展，是RBAC的角色分层模型，RBAC1引入了角色继承概念，有了继承就有了上下级的包含关系。

![1566811977451](F:\GitDepository\学习文档\Spring Cloud\baseImages\RBAC1.png)

---

##### c、RBAC2，是RBAC的约束模型

