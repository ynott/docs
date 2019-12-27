---
title: "クラスタデータストアオプション"
weight: 50
---

etcd以外のデータストアを使ってKubernetesを実行する機能は他のKubernetesディストリビューションとK3sが大きく違うところです。この機能によりKubernetesを柔軟な運用を可能にします。様々なデータストアを使えることによりユースケースに最適なデータストアを選ぶことができます。

* チームにetcdの操作に関する専門知識がない場合は、MySQLやPostgreSQLなどのエンタープライズグレードのSQLデータベースを利用できます。
* シンプルで生存期間の短いクラスタをCI/CD環境で実行する必要がある場合は、組み込みSQLiteデータベースを使用できます。
* エッジにKubernetesをデプロイし、高可用性ソリューションを必要としているが、エッジでデータベースを管理する運用上のオーバーヘッドを許容できない場合、DQLiteで構築されたK3sの組み込みHAデータストアを使用できます(現在試験中)。

K3は、オプションとして以下のデータストアをサポートします:

* 組み込み [SQLite](https://www.sqlite.org/index.html)
* [PostgreSQL](https://www.postgresql.org/) (認定されているバージョン 10.7 and 11.5)
* [MySQL](https://www.mysql.com/) (認定されているバージョン 5.7)
* [etcd](https://etcd.io/) (認定されているバージョン 3.3.15)
* 高可用性組み込み [DQLite](https://dqlite.io/) (試験実装)

### 外部データストア構成パラメータ
PostgreSQL、MySQL、etcdなどの外部データストアを使用する場合は、K3sが接続方法を認識できるように `datastore-endpoint` パラメータを設定する必要があります。パラメータを指定して、接続の認証と暗号化を構成することもできます。次の表は、CLIフラグまたは環境変数として渡すことができるこれらのパラメータをまとめたものです。

  CLI フラグ | 環境変数 | 詳細
  ------------|-------------|------------------
 <span style="white-space: nowrap">`--datastore-endpoint`</span> | `K3S_DATASTORE_ENDPOINT` | PostgresSQL、MySQL、またはetcd接続文字列を指定します。これは、データストアへの接続を記述するために使用される文字列です。この文字列の構造は各バックエンドに固有で、以下に詳細を示します。|
 <span style="white-space: nowrap">`--datastore-cafile`</span> | `K3S_DATASTORE_CAFILE` | データストアとの通信のセキュリティ保護に役立つTLS Certificate Authority(CA)ファイル。データストアが、カスタム認証局によって署名された証明書を使用してTLS経由で要求を処理する場合、K3sクライアントが証明書を正しく検証できるように、このパラメータを使用してCAを指定できます。|                              
|  <span style="white-space: nowrap">`--datastore-certfile`</span> | `K3S_DATASTORE_CERTFILE` | データストアへのクライアント証明書ベースの認証に使用されるTLS証明書ファイル。この機能を使用するには、クライアント証明書ベースの認証をサポートするようにデータストアを構成する必要があります。このパラメーターを指定する場合は、`datastore-keyfile`パラメーターも指定する必要があります。|     
|  <span style="white-space: nowrap">`--datastore-keyfile`</span> | `K3S_DATASTORE_KEYFILE` | データストアへのクライアント証明書ベースの認証に使用されるTLSキーファイル。詳細については、前述の `datastore-certfile` パラメータを参照してください。|

データベース資格情報やその他の機密情報がプロセス情報の一部として公開されないように、これらのパラメータをコマンドライン引数ではなく環境変数として設定することをお勧めします。

### Datastore Endpoint Format and Functionality
As mentioned, the format of the value passed to the `datastore-endpoint` parameter is dependent upon the datastore backend. The following details this format and functionality for each supported external datastore.

{{% tabs %}}
{{% tab "PostgreSQL" %}}

In its most common form, the datastore-endpoint parameter for PostgreSQL has the following format:

`postgres://username:password@hostname:port/database-name`

More advanced configuration parameters are available. For more information on these, please see https://godoc.org/github.com/lib/pq.

If you specify a database name and it does not exist, the server will attempt to create it.

If you only supply `postgres://`  as the endpoint, K3s will attempt to do the following:

* Connect to localhost using `postgres` as the username and password
* Create a database named `kubernetes`


{{% /tab %}}
{{% tab "MySQL" %}}

In its most common form, the `datastore-endpoint` parameter for MySQL has the following format:

`mysql://username:password@tcp(hostname:3306)/database-name`

More advanced configuration parameters are available. For more information on these, please see https://github.com/go-sql-driver/mysql#dsn-data-source-name

Note that due to a [known issue](https://github.com/rancher/k3s/issues/1093) in K3s, you cannot set the `tls` parameter. TLS communication is supported, but you cannot, for example, set this parameter to "skip-verify" to cause K3s to skip certificate verification.

If you specify a database name and it does not exist, the server will attempt to create it.

If you only supply `mysql://` as the endpoint, K3s will attempt to do the following:

* Connect to the MySQL socket at `/var/run/mysqld/mysqld.sock` using the `root` user and no password
* Create a database with the name `kubernetes`


{{% /tab %}}
{{% tab "etcd" %}}

In its most common form, the `datastore-endpoint` parameter for etcd has the following format:

`https://etcd-host-1:2379,https://etcd-host-2:2379,https://etcd-host-3:2379`

The above assumes a typical three node etcd cluster. The parameter can accept one more comma separated etcd URLs.

{{% /tab %}}
{{% /tabs %}}

<br/>Based on the above, the following example command could be used to launch a server instance that connects to a PostgresSQL database named k3s:
```
K3S_DATASTORE_ENDPOINT='postgres://username:password@hostname:5432/k3s' k3s server
```

And the following example could be used to connect to a MySQL database using client certificate authentication:
```
K3S_DATASTORE_ENDPOINT='mysql://username:password@tcp(hostname:3306)/k3s' \
K3S_DATASTORE_CERTFILE='/path/to/client.crt' \
K3S_DATASTORE_KEYFILE='/path/to/client.key' \
k3s server
```

### Embedded DQLite for HA (Experimental)
K3s's use of DQLite is similar to its use of SQLite. It is simple to setup and manage. As such, there is no external configuration or additional steps to take in order to use this option. Please see [High Availability with Embedded DB (Experimental)]({{< baseurl >}}/k3s/latest/en/installation/ha-embedded/) for instructions on how to run with this option.
