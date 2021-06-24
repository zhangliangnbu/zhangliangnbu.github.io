---
title: Java-Spring-Boot入门
tags:
- java
- spring
categories:
- java
---



```sql
/**楼层配置表*/
CREATE TABLE `sales_config` (
`sys_no` bigint(20) unsigned NOT NULL COMMENT '系统编号',
`parent_syn_no` bigint(20) unsigned NOT NULL COMMENT '父级分类系统编号 0-顶级父节点',
`business_type` tinyint(4) NOT NULL COMMENT '业务类型 1-首页企业智选 2-首页场景 3-首页分类 4-首页品牌 5-二级页面场景 6-二级页面场景促销商品分类', 
`name` varchar(130) NOT NULL COMMENT '名称',
`description` varchar(200) DEFAULT NULL COMMENT '描述',
`image` varchar(260) DEFAULT NULL COMMENT '图片地址',
`sort_index` tinyint(4) NOT NULL COMMENT '分类排序',
`url` varchar(260) DEFAULT NULL COMMENT '跳转链接',
`in_user_sys_no` bigint(20) NOT NULL COMMENT '创建者系统编号',
`in_user_name` varchar(40) NOT NULL COMMENT '创建人',
`in_date` datetime NOT NULL COMMENT '创建日期',
`edit_user_sys_no` bigint(20) NOT NULL COMMENT '编辑人系统编号',
`edit_user_name` varchar(40) NOT NULL COMMENT '编辑人',
`edit_date` datetime NOT NULL COMMENT '编辑日期',
`common_status` int(11) NOT NULL COMMENT '数据状态0-无效 1-有效 -999-删除',
PRIMARY KEY (`sys_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='楼层配置表';

/**促销商品表*/
CREATE TABLE `sales_config_product` (
`sys_no` bigint(20) unsigned NOT NULL COMMENT '系统编号',
`parent_sys_no` bigint(20) unsigned NOT NULL COMMENT '商品所属的父级分类系统编号',
`sku_sys_no` bigint(20) unsigned NOT NULL COMMENT '商品系统编号',
`sku_sold_count` bigint(20) unsigned NOT NULL COMMENT '商品出售数量',
`isHot` boolean NOT NULL COMMENT '是否为热销商品: true-热销，false-非热销',
`sort_index` tinyint(4) NOT NULL COMMENT '分类排序',
`in_user_sys_no` bigint(20) NOT NULL COMMENT '创建者系统编号',
`in_user_name` varchar(40) NOT NULL COMMENT '创建人',
`in_date` datetime NOT NULL COMMENT '创建日期',
`edit_user_sys_no` bigint(20) NOT NULL COMMENT '编辑人系统编号',
`edit_user_name` varchar(40) NOT NULL COMMENT '编辑人',
`edit_date` datetime NOT NULL COMMENT '编辑日期',
`common_status` int(11) NOT NULL COMMENT '数据状态0-无效 1-有效 -999-删除',
PRIMARY KEY (`sys_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='促销商品表 用于楼层和专题';

/**主题促销表*/
CREATE TABLE `sales_topic` (
`sys_no` bigint(20) unsigned NOT NULL COMMENT '系统编号',
`topic_id` bigint(20) unsigned NOT NULL COMMENT '主题id',
`topic_name` varchar(130) DEFAULT NULL COMMENT '主题名称',
`in_user_sys_no` bigint(20) NOT NULL COMMENT '创建者系统编号',
`in_user_name` varchar(40) NOT NULL COMMENT '创建人',
`in_date` datetime NOT NULL COMMENT '创建日期',
`edit_user_sys_no` bigint(20) NOT NULL COMMENT '编辑人系统编号',
`edit_user_name` varchar(40) NOT NULL COMMENT '编辑人',
`edit_date` datetime NOT NULL COMMENT '编辑日期',
`common_status` int(11) NOT NULL COMMENT '数据状态0-无效 1-有效 -999-删除',
PRIMARY KEY (`sys_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='主题促销信息';

/**主题和Banner关系表*/
CREATE TABLE `sales_topic_banner_relation` (
`sys_no` bigint(20) unsigned NOT NULL COMMENT '系统编号',
`topic_id` bigint(20) unsigned NOT NULL COMMENT '主题id',
`banner_sys_no` bigint(20) unsigned NOT NULL COMMENT 'Banner系统编号',
`in_user_sys_no` bigint(20) NOT NULL COMMENT '创建者系统编号',
`in_user_name` varchar(40) NOT NULL COMMENT '创建人',
`in_date` datetime NOT NULL COMMENT '创建日期',
`edit_user_sys_no` bigint(20) NOT NULL COMMENT '编辑人系统编号',
`edit_user_name` varchar(40) NOT NULL COMMENT '编辑人',
`edit_date` datetime NOT NULL COMMENT '编辑日期',
`common_status` int(11) NOT NULL COMMENT '数据状态0-无效 1-有效 -999-删除',
PRIMARY KEY (`sys_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='主题Banner关系';
```





