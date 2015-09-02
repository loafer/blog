Zookeeper —— ACL
==

Zookeeper使用ACL控制节点的访问。ACL与Unix的文件访问权限相似，通过权限位控制节点的操作。但不同的是，节点没有被限制在用户（所有者）、组、其他。注意Zookeeper没有所有者的概念。

ACL只属于一个节点，不会被传递给子节点。例如：/app只对ip:172.16.16.1可读，而/app/staus对任何人都可读。

Zookeeper支持插入式的验证方案。Ids使用如下形式 `scheme:id`。其中scheme是id所对应的认证方案。例如，`ip:172.16.16.1`是一个针对主机地址是172.16.16.1的id。

当客户端连接到ZooKeeper验证自己时，ZooKeeper将有关该客户端的所有Id与客户连接关联。当客户端访问节点时，节点上的ACLs会再次检查。ACLs由（scheme:expression, perms）对组成。表达式格式是特定的。例如：(ip:19.22.0.0/16, READ) 表示任何一个ip以19.22开头的客户端都有读权限。

##ACL权限
Zookeeper支持下列权限：

- **CREATE:** 你可以创建节点

- **READ: ** 你可以获得节点数据和他的子节点

- **WRITE: ** 你可以为节点设置数据

- **DELETE: ** 你可以删除一个子节点

- **ADMIN: ** 你可以设置权限

 
CREATE 和DELETE已经从WRITE分离出来，目的是为了更好的访问控制。CREATE 和DELETE适用以下场合：
具有CREATE但无DELETE权限：客户端发出创建请求，是在父目录下创建创建节点，你想让所有的客户能添加节点，但只有创建的申请者能删除（这类似于文件的APPEND权限）

因为Zookeeper没有文件所有者的概念，所以有些场景下ADMIN权限就相当所有者权限。Zookeeper不支持LOOKUP权限，每个人隐含有LOOKUP权限。它允许你查看节点状态，单不能做其它事（这就存在一个问题，如果对一个不存在的节点调用zoo_exists()，不会进行安全检查）。

**内置ACL方案**
Zookeeper有下列内建方案：

- **word**  使用唯一id标识`anyone`，代表所有人。

- **auth** 不使用任何id，代表已认证用户

- **digest** 使用`username:password`生成MD5串作为ACL ID标识。明文发送，ACL表达式中使用的是`username:base64`，base64是password编码后的摘要。

- **ip** 使用客户端主机IP作为ACL ID标识。ACL表达式格式`addr/bits`，此时addr中的有效位与客户端addr中的有效位进行比对。
