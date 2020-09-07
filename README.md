# ClickHouse Operator

ClickHouse Operator 创建，配置和管理在 Kubernetes上运行的ClickHouse集群。


**重要: 如果将Operator从0.6.0或更早版本升级到0.7.0或更高版本，请确保您的ClickHouseInstallation名称短于15个符号。 否则，请勿升级操作员。 如果您有升级问题，请联系support@altinity.com。**

[![issues](https://img.shields.io/github/issues/altinity/clickhouse-operator.svg)](https://github.com/altinity/clickhouse-operator/issues)
[![tags](https://img.shields.io/github/tag/altinity/clickhouse-operator.svg)](https://github.com/altinity/clickhouse-operator/tags)
[![Go Report Card](https://goreportcard.com/badge/github.com/altinity/clickhouse-operator)](https://goreportcard.com/report/github.com/altinity/clickhouse-operator)
[![Docker Pulls](https://img.shields.io/docker/pulls/altinity/clickhouse-operator.svg)](https://hub.docker.com/r/altinity/clickhouse-operator)
[![CircleCI](https://circleci.com/gh/Altinity/clickhouse-operator.svg?style=svg)](https://circleci.com/gh/Altinity/clickhouse-operator)


## Features 特征

Kubernetes 上的 ClickHouse Operator 目前提供以下内容：

- 创建Clickhouse 集群 根据自定义资源 [specification/规范][chi_max_yaml]
- 定制的存储配置（VolumeClaim模板）
- 定制 pod 模板
- 定制 server 模板 从 endpoints
- ClickHouse配置和设置（包括Zookeeper集成）
- 灵活的模板
- ClickHouse群集扩展 包括schema自动传播
- ClickHouse版本升级
- 将 ClickHouse 指标导出到 Prometheuss

## Requirements/要求

 * Kubernetes 1.15.11+
 
## Documentation/文献资料

[快速入门指南][quick_start_guide]

**Advanced setups**
 * [详细的 Operator 安装说明][detailed_installation_instructions]
   * [Operator 配置][operator_configuration]
 * [设置 ClickHouse 集群 用于复制][replication_setup]
   * [配置 Zookeeper][zookeeper_setup]
 * [配置永久存储][storage_configuration]
 * [ClickHouse安装自定义资源规范][crd_explained]
 
**维护任务**
 * [基于 ClickHouse 集群添加 replication 功能][update_cluster_add_replication]
 * [Schema 维护][schema_migration]
 * [更新ClickHouse版本][update_clickhouse_version]
 * [更新 Operator 版本][update_operator]

**Monitoring/监控方式**
 * [设置监控][monitoring_setup]
 * [Prometheus和Clickhouse-operator集成][prometheus_setup]
 * [Grafana和Prometheus整合][grafana_setup]

**How to contribute**
 * [How to contribute/submit a patch][contributing_manual]
 
---
**All docs**
 * [所有可用的文档列表][all_docs_list]
---
 
## License

Copyright (c) 2019-2020, Altinity Ltd and/or its affiliates. All rights reserved.

`clickhouse-operator` is licensed under the Apache License 2.0.

See [LICENSE](./LICENSE) for more details.
 
[chi_max_yaml]: ./docs/chi-examples/99-clickhouseinstallation-max.yaml
[intro]: ./docs/introduction.md
[quick_start_guide]: ./docs/quick_start.md
[detailed_installation_instructions]: ./docs/operator_installation_details.md
[replication_setup]: ./docs/replication_setup.md
[crd_explained]: ./docs/custom_resource_explained.md
[zookeeper_setup]: ./docs/zookeeper_setup.md
[monitoring_setup]: ./docs/monitoring_setup.md
[prometheus_setup]: ./docs/prometheus_setup.md
[grafana_setup]: ./docs/grafana_setup.md
[storage_configuration]: ./docs/storage.md
[update_cluster_add_replication]: ./docs/chi_update_add_replication.md
[update_clickhouse_version]: ./docs/chi_update_clickhouse_version.md
[update_operator]: ./docs/operator_upgrade.md
[schema_migration]: ./docs/schema_migration.md
[operator_configuration]: ./docs/operator_configuration.md
[all_docs_list]: ./docs/README.md
[contributing_manual]: ./CONTRIBUTING.md
