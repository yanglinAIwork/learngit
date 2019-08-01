## Docker下启动Nacos

### 1、查找镜像

​	https://hub.docker.com/r/nacos/nacos-server

### 2、下载镜像

~~~
docker pull nacos/nacos-server
~~~

### 3、查看镜像

| REPOSITORY         | TAG    | IMAGE ID     | CREATED     | SIZE  |      |
| ------------------ | ------ | ------------ | ----------- | ----- | ---- |
| nacos/nacos-server | latest | fbf2aa0a0435 | 5 weeks ago | 697MB |      |

### 4、运行镜像（单机模式）

~~~
docker run -d \
--name nacos-server \
--network common-network \
-e PREFER_HOST_MODE=hostname \
-e MODE=standalone \
--add-host localhost:10.63.203.39
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_MASTER_SERVICE_HOST=mysql-master \
-e MYSQL_MASTER_SERVICE_PORT=3306 \
-e MYSQL_MASTER_SERVICE_USER=root \
-e MYSQL_MASTER_SERVICE_PASSWORD=123456 \
-e MYSQL_MASTER_SERVICE_DB_NAME=nacos \
-e MYSQL_SLAVE_SERVICE_HOST=mysql-slave \
-e MYSQL_SLAVE_SERVICE_PORT=3306 \
-p 8848:8848 \
nacos/nacos-server
~~~

当前启动命令

~~~
docker run -d --name nacos-server -e PREFER_HOST_MODE=hostname -e MODE=standalone --add-host localhost:10.63.203.39 -p 8848:8848 nacos/
nacos-server
~~~

**访问地址：**

​	**http://127.0.0.1:8848/nacos/index.html#/login**

​	**默认账号密码是nacos/nacos**

---

**配置项参数：**

| 配置项                        | 描述                    | 可选参数           | 默认值  |
| ----------------------------- | ----------------------- | ------------------ | ------- |
| MODE                          | 模式 cluster/standalone | cluster/standalone | cluster |
| PREFER_HOST_MODE              | 是否支持 hostname       | hostname/ip        | ip      |
| NACOS_SERVER_PORT             | 服务端口号              |                    | 8848    |
| SPRING_DATASOURCE_PLATFORM    | 单机模式支持 mysql      | mysql / empty      | empty   |
| MYSQL_MASTER_SERVICE_HOST     | mysql 主节点 host       |                    |         |
| MYSQL_MASTER_SERVICE_PORT     | mysql 主节点 port       |                    | 3306    |
| MYSQL_MASTER_SERVICE_DB_NAME  | mysql 主节点数据库名    |                    |         |
| MYSQL_MASTER_SERVICE_USER     | mysql 主节点用户名      |                    |         |
| MYSQL_MASTER_SERVICE_PASSWORD | mysql 主节点密码        |                    |         |
| MYSQL_SLAVE_SERVICE_HOST      | mysql 从节点 host       |                    |         |
| MYSQL_SLAVE_SERVICE_PORT      | mysql 从节点 port       |                    | 3306    |

### 5、配置数据库

~~~
cd config
vi application.properties
~~~

![img](D:\Typora文档\文档\学习文档\Docker\DockerNacosMysql.png)

注：把db.url.1这段配置注释掉

出现的问题：

~~~
Caused by: java.sql.SQLException: null,  message from server: "Host 'host.docker.internal' is not allowed to connect to this MySQL server"
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:996)
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:935)
        at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:924)
        at com.mysql.jdbc.MysqlIO.doHandshake(MysqlIO.java:1037)
        at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2234)
        at com.mysql.jdbc.ConnectionImpl.connectWithRetries(ConnectionImpl.java:2085)
~~~

解决方法：

​	mysql服务器出于安全考虑，默认只允许本地登录数据库服务器。

~~~
mysql -u root -pvmware
mysql>use mysql;
update user set host ='%' where user ='root';
flush privileges;
select host, user from user;
quit;

之后重启Mysql
或者管理员命令 net stop mysql /net start mysql
~~~

---

## 附录

### 建表语句

~~~sql
/*
Navicat MySQL Data Transfer
Source Server         : cenos
Source Server Version : 50642
Source Host           : 192.168.247.131:3306
Source Database       : nacos_config
Target Server Type    : MYSQL
Target Server Version : 50642
File Encoding         : 65001
Date: 2019-04-18 16:42:02
*/
 
SET FOREIGN_KEY_CHECKS=0;
 
-- ----------------------------
-- Table structure for config_info
-- ----------------------------
DROP TABLE IF EXISTS `config_info`;
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) COLLATE utf8_bin DEFAULT NULL,
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(20) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) COLLATE utf8_bin DEFAULT NULL,
  `c_use` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `effect` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `type` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `c_schema` text COLLATE utf8_bin,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
 
-- ----------------------------
-- Records of config_info
-- ----------------------------
INSERT INTO `config_info` VALUES ('3', 'springcloud-sys-dev.yaml', 'DEFAULT_GROUP', 0x64656D6F3A0D0A2020746573743A2074657374, '395d7c0385d65852419ff666c131e7a6', '2019-04-18 03:16:51', '2019-04-18 03:20:20', null, '192.168.247.1', '', '', '', null, null, 'yaml', null);
 
-- ----------------------------
-- Table structure for config_info_aggr
-- ----------------------------
DROP TABLE IF EXISTS `config_info_aggr`;
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'datum_id',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
 
-- ----------------------------
-- Records of config_info_aggr
-- ----------------------------
 
-- ----------------------------
-- Table structure for config_info_beta
-- ----------------------------
DROP TABLE IF EXISTS `config_info_beta`;
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) COLLATE utf8_bin DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(20) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
 
-- ----------------------------
-- Records of config_info_beta
-- ----------------------------
 
-- ----------------------------
-- Table structure for config_info_tag
-- ----------------------------
DROP TABLE IF EXISTS `config_info_tag`;
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(20) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
 
-- ----------------------------
-- Records of config_info_tag
-- ----------------------------
 
-- ----------------------------
-- Table structure for config_tags_relation
-- ----------------------------
DROP TABLE IF EXISTS `config_tags_relation`;
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
 
-- ----------------------------
-- Records of config_tags_relation
-- ----------------------------
 
-- ----------------------------
-- Table structure for group_capacity
-- ----------------------------
DROP TABLE IF EXISTS `group_capacity`;
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';
 
-- ----------------------------
-- Records of group_capacity
-- ----------------------------
 
-- ----------------------------
-- Table structure for his_config_info
-- ----------------------------
DROP TABLE IF EXISTS `his_config_info`;
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL,
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL,
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL,
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` text COLLATE utf8_bin,
  `src_ip` varchar(20) COLLATE utf8_bin DEFAULT NULL,
  `op_type` char(10) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
 
-- ----------------------------
-- Records of his_config_info
-- ----------------------------
INSERT INTO `his_config_info` VALUES ('0', '1', 'springcloud-sys-dev.ymal', 'DEFAULT_GROUP', '', 0x64656D6F3A0D0A202020737472696E673A2064656D6F, '0308187c93530d5da547cb86b9a35203', '2010-05-05 00:00:00', '2019-04-18 03:08:06', null, '192.168.247.1', 'I', '');
INSERT INTO `his_config_info` VALUES ('1', '2', 'springcloud-sys-dev.ymal', 'DEFAULT_GROUP', '', 0x64656D6F3A0D0A202020737472696E673A2064656D6F, '0308187c93530d5da547cb86b9a35203', '2010-05-05 00:00:00', '2019-04-18 03:10:52', null, '192.168.247.1', 'U', '');
INSERT INTO `his_config_info` VALUES ('0', '3', 'springcloud-sys-dev.yaml', 'DEFAULT_GROUP', '', 0x6A77743A0D0A2020746F6B656E2D6865616465723A20417574686F72697A6174696F6E, 'b9703f4b7560c8c21645a7193a85885e', '2010-05-05 00:00:00', '2019-04-18 03:16:51', null, '192.168.247.1', 'I', '');
INSERT INTO `his_config_info` VALUES ('1', '4', 'springcloud-sys-dev.ymal', 'DEFAULT_GROUP', '', 0x6A77743A0D0A2020746F6B656E2D6865616465723A20417574686F72697A6174696F6E, 'b9703f4b7560c8c21645a7193a85885e', '2010-05-05 00:00:00', '2019-04-18 03:16:59', null, '192.168.247.1', 'D', '');
INSERT INTO `his_config_info` VALUES ('3', '5', 'springcloud-sys-dev.yaml', 'DEFAULT_GROUP', '', 0x6A77743A0D0A2020746F6B656E2D6865616465723A20417574686F72697A6174696F6E, 'b9703f4b7560c8c21645a7193a85885e', '2010-05-05 00:00:00', '2019-04-18 03:20:20', null, '192.168.247.1', 'U', '');
 
-- ----------------------------
-- Table structure for roles
-- ----------------------------
DROP TABLE IF EXISTS `roles`;
CREATE TABLE `roles` (
  `username` varchar(50) NOT NULL,
  `role` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
-- ----------------------------
-- Records of roles
-- ----------------------------
INSERT INTO `roles` VALUES ('nacos', 'ROLE_ADMIN');
 
-- ----------------------------
-- Table structure for tenant_capacity
-- ----------------------------
DROP TABLE IF EXISTS `tenant_capacity`;
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
 
-- ----------------------------
-- Records of tenant_capacity
-- ----------------------------
 
-- ----------------------------
-- Table structure for tenant_info
-- ----------------------------
DROP TABLE IF EXISTS `tenant_info`;
CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) COLLATE utf8_bin DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
 
-- ----------------------------
-- Records of tenant_info
-- ----------------------------
 
-- ----------------------------
-- Table structure for users
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(500) NOT NULL,
  `enabled` tinyint(1) NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
-- ----------------------------
-- Records of users
-- ----------------------------
INSERT INTO `users` VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', '1');


~~~

