# `clickhouse-operator` configuration

## 介绍

`clickhouse-operator` 可以通过多种方式进行配置。配置包括以下主要部分:

1. 操作员设置——操作员设置控制操作员本身的行为。
1. ClickHouse通用配置文件-即用的XML文件，带有ClickHouse配置**原样**的部分。
常见配置通常包含一般的ClickHouse配置部分，如网络侦听端点、日志程序选项等。这些都是通过配置映射公开的。
1. ClickHouse用户配置文件-随时可以使用的XML文件，带有ClickHouse配置的部分
用户配置通常包含带有用户帐户规范的ClickHouse配置部分。这些也通过配置映射公开。
1. “ClickHouseOperatorConfiguration”资源。
1. ClickHouseInstallationTemplate的。操作符提供了将' ClickHouseInstallation '清单的部分指定为一组模板的功能，这将在所有' ClickHouseInstallation '中使用。

## Operator 设置

Operator 设置初始化顺序从3个来源:
* `/etc/clickhouse-operator/config.yaml`
* etc-clickhouse-operator-files configmap (also a part of default [clickhouse-operator-install.yaml][clickhouse-operator-install.yaml]
* `ClickHouseOperatorConfiguration` resource. See [example][70-chop-config.yaml] for details.

Next sources merges with the previous one. Changes to `etc-clickhouse-operator-files` are not monitored, but picked up if operator is restarted. Changes to `ClickHouseOperatorConfiguration` are monitored by an operator and applied immediately.

下一个源与前一个源合并。对etc-clickhouse- operatoror文件的更改不会被监视，但如果操作员重新启动就会被拾取。对ClickHouseOperatorConfiguration的更改由操作员监控并立即应用。

`config.yaml` has following settings:

```yaml
################################################
##
## Watch Namespaces Section
##
################################################

# List of namespaces where clickhouse-operator watches for events.
# Concurrently running operators should watch on different namespaces
# watchNamespaces:
#  - dev
#  - info
#  - onemore

################################################
##
## Additional Configuration Files Section
##
################################################

# Path to folder where ClickHouse configuration files common for all instances within CHI are located.
chCommonConfigsPath: config.d

# Path to folder where ClickHouse configuration files unique for each instance (host) within CHI are located.
chHostConfigsPath: conf.d

# Path to folder where ClickHouse configuration files with users settings are located.
# Files are common for all instances within CHI
chUsersConfigsPath: users.d

# Path to folder where ClickHouseInstallation .yaml manifests are located.
# Manifests are applied in sorted alpha-numeric order
chiTemplatesPath: templates.d

################################################
##
## Cluster Create/Update/Delete Objects Section
##
################################################

# How many seconds to wait for created/updated StatefulSet to be Ready
statefulSetUpdateTimeout: 600

# How many seconds to wait between checks for created/updated StatefulSet status
statefulSetUpdatePollPeriod: 10

# What to do in case created StatefulSet is not in Ready after `statefulSetUpdateTimeout` seconds
# Possible options:
# 1. abort - do nothing, just break the process and wait for admin
# 2. delete - delete newly created problematic StatefulSet
onStatefulSetCreateFailureAction: delete

# What to do in case updated StatefulSet is not in Ready after `statefulSetUpdateTimeout` seconds
# Possible options:
# 1. abort - do nothing, just break the process and wait for admin
# 2. rollback - delete Pod and rollback StatefulSet to previous Generation.
# Pod would be recreated by StatefulSet based on rollback-ed configuration
onStatefulSetUpdateFailureAction: rollback

################################################
##
## ClickHouse Settings Section
##
################################################

# Default values for ClickHouse user configuration
# 1. user/profile - string
# 2. user/quota - string
# 3. user/networks/ip - multiple strings
# 4. user/password - string
chConfigUserDefaultProfile: default
chConfigUserDefaultQuota: default
chConfigUserDefaultNetworksIP:
  - "::/0"
chConfigUserDefaultPassword: "default"

################################################
##
## Operator's access to ClickHouse instances
##
################################################

# ClickHouse credentials (username, password and port) to be used by operator to connect to ClickHouse instances for:
# 1. Metrics requests
# 2. Schema maintenance
# 3. DROP DNS CACHE
# User with such credentials credentials can be specified in additional ClickHouse .xml config files,
# located in `chUsersConfigsPath` folder
chUsername: clickhouse_operator
chPassword: clickhouse_operator_password
chPort: 8123
```

## ClickHouse Installation settings

Operator deploys ClickHouse clusters with different defaults, that can be configured in a flexible way. 

Operator使用不同的缺省值部署ClickHouse集群，可以以灵活的方式进行配置

### 默认的ClickHouse配置文件

Default ClickHouse configuration files can be found in the following config maps, that are mounted to corresponding configuration folders of ClickHouse pods:

默认的ClickHouse配置文件可以在下面的配置映射中找到，它们被挂载到ClickHouse pods相应的配置文件夹中:

* etc-clickhouse-operator-confd-files
* etc-clickhouse-operator-configd-files
* etc-clickhouse-operator-usersd-files

默认情况下初始化配置映射 [clickhouse-operator-install.yaml][clickhouse-operator-install.yaml].

### Defaults for ClickHouseInstallation

默认的ClickHouseInstallationTemplate可以提供它的多种方式:
* etc-clickhouse-operator-templatesd-files configmap
* `ClickHouseInstallationTemplate` resources.

`ClickHouseInstallationTemplate` has the same structure as `ClickHouseInstallation`, but all parts and fields are optional. Templates are included into an installation with 'useTemplates' syntax. For example, one can define a template for ClickHouse pod:

ClickHouseInstallationTemplate与ClickHouseInstallation具有相同的结构，但是所有的部件和字段都是可选的。模板包含在带有“useTemplates”语法的安装中。例如，可以为ClickHouse pod定义一个模板

```apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallationTemplate"

metadata:
  name: clickhouse-stable

spec:
  templates:
    podTemplates:
      - name: default
        spec:
          containers:
            - name: clickhouse-pod
              image: yandex/clickhouse-server:19.11.8.46
```

Template needs to be deployed to some namespace, and later on used in the installation:

模板需要部署到某个名称空间，稍后在安装中使用:

```
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
...
spec:
  useTemplates:
    - name: clickhouse-stable
...
```

[clickhouse-operator-install.yaml]: ../deploy/operator/clickhouse-operator-install.yaml
[70-chop-config.yaml]: ./chi-examples/70-chop-config.yaml
