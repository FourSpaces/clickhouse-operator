
# Setup ClickHouse cluster with data replication

## 先决条件

1. ClickHouse operator [installed][operator_installation_details.md]
1. Zookeeper [installed][zookeeper_setup.md]


## 清单

Let's take a look on [example][chi-examples/04-replication-zookeeper-05-simple-PV.yaml], which creates a cluster with 2 shards and 2 replicas and persistent storage.

让我们看一下示例，它创建了一个具有2个碎片、2个副本和持久存储的集群。

```yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"

metadata:
  name: "repl-05"

spec:
  defaults:
    templates: 
      dataVolumeClaimTemplate: default
      podTemplate: clickhouse:19.6
 
  configuration:
    zookeeper:
      nodes:
      - host: zookeeper.zoo1ns
    clusters:
      - name: replicated
        layout:
          shardsCount: 2
          replicasCount: 2

  templates:
    volumeClaimTemplates:
      - name: default
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Mi
    podTemplates:
      - name: clickhouse:19.6
        spec:
          containers:
            - name: clickhouse-pod
              image: yandex/clickhouse-server:19.6.2.11
```


## 复制表设置

### Macros
Operator provides set of [macros][macros], which are:

操作符提供一组宏，它们是:

 1. `{installation}` -- ClickHouse Installation name
 1. `{cluster}` -- 主集群名称
 1. `{replica}` -- 集群中的副本名称映射到pod服务名称
 1. `{shard}` -- shard id

ClickHouse also supports internal macros `{database}` and `{table}` that maps to current **database** and **table** respectively.

ClickHouse还支持分别映射到当前数据库和表的内部宏{database}和{table}。



### Create replicated table

Now we can create [replicated table][replication], using specified macros

现在我们可以使用指定的宏创建复制表

```sql
CREATE TABLE events_local on cluster '{cluster}' (
    event_date  Date,
    event_type  Int32,
    article_id  Int32,
    title       String
  ) engine=ReplicatedMergeTree('/clickhouse/{installation}/{cluster}/tables/{shard}/{database}/{table}', '{replica}', event_date, (event_type, article_id), 8192);
```

```sql
CREATE TABLE events on cluster '{cluster}' AS events_local
ENGINE = Distributed('{cluster}', default, events_local, rand());
```

我们可以生成一些数据:
```sql
INSERT INTO events SELECT today(), rand()%3, number, 'my title' FROM numbers(100);
```

并检查这些数据如何在集群中分布
```sql
SELECT count() FROM events;
SELECT count() FROM events_local;
```

[operator_installation_details.md]: ./operator_installation_details.md
[zookeeper_setup.md]: ./zookeeper_setup.md
[chi-examples/04-replication-zookeeper-05-simple-PV.yaml]: ./chi-examples/04-replication-zookeeper-05-simple-PV.yaml
[macros]: https://clickhouse.yandex/docs/en/operations/server_settings/settings/#macros
[replication]: https://clickhouse.yandex/docs/en/operations/table_engines/replication/
