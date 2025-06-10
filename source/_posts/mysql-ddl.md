---
title: mysql 常用ddl
date: 2025-06-10 17:12:50
tags:
---

## 创建表格

```sql
CREATE TABLE `report_generate_task_record` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `taskId` varchar(255) NOT NULL COMMENT '任务id',
  `source` varchar(255) NOT NULL COMMENT '业务线',
  `sceneId` bigint NOT NULL COMMENT '场景id',
  `createTime` bigint NOT NULL COMMENT '创建时间',
  `modifyTime` bigint NOT NULL COMMENT '修改时间',
  `startTime` bigint DEFAULT NULL COMMENT '内部场景-开始时间',
  `endTime` bigint DEFAULT NULL COMMENT '内部场景-结束时间',
  `dataSource` longtext COMMENT '外部场景-数据源',
  `status` int NOT NULL DEFAULT '0' COMMENT '任务状态',
  `reportId` char(36) DEFAULT NULL COMMENT '简报id',
  `reportHtml` longtext COMMENT '报告html',
  `combine_report_id` bigint DEFAULT NULL COMMENT '组合简报id',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=98 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

## 表添加一列

```sql
use ireport;
alter table report_exec_record 
    add column combine_report_id bigint DEFAULT NULL COMMENT '组合简报id';
```

## 表添加多列

```sql
use ireport;
alter table report_generate_task_record 
    add column reportPic longtext DEFAULT NULL COMMENT '报告pic',
    add column picCost bigint DEFAULT NULL COMMENT '图片生成耗时';
```