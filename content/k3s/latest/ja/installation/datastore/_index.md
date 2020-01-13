---
title: "クラスタデータストアオプション"
weight: 50
---

他のKubernetesディストリビューションとK3sが大きく違うところは、etcd以外のデータストアを使ってKubernetesを実行する機能です。この機能によりKubernetesの柔軟な運用ができるようになりました。様々なデータストアを使えることによりユースケースに最適なデータストアを選ぶことができます。例を挙げると:

* チームにetcdの操作に関する専門知識がない場合は、MySQLやPostgreSQLなどのエンタープライズグレードのSQLデータベースを利用できます。
* シンプルで生存期間の短いクラスタをCI/CD環境で実行する必要がある場合は、組み込みSQLiteデータベースを使用できます。
* エッジにKubernetesをデプロイし、高可用性ソリューションを必要としているが、エッジでデータベースを管理する運用上のオーバーヘッドを許容できない場合、DQLiteで構築されたK3sの組み込みHAデータストアを使用できます(現在、試験実装)。

K3sは、オプションとして以下のデータストアをサポートします:

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

### データストアのエンドポイントの形式と機能
前述のように、`datastore-endpoint`パラメータに渡される値の形式は、データストアバックエンドに依存します。次に、それぞれサポートされている外部データストアの形式と機能の詳細を示します。

{{% tabs %}}
{{% tab "PostgreSQL" %}}

最も一般的な形式では、PostgreSQLのdatastore-endpointパラメータは以下の形式です:

`postgres://username:password@hostname:port/database-name`

より高度な設定パラメータを使用できます。詳細については、https://godoc.org/github.com/lib/pq を参照してください。

指定したデータベース名が存在しない場合、サーバーはそのデータベース名を作成しようとします。

エンドポイントとして `postgres://` のみを指定すると、K3sは以下のことを実行しようとします:

* ユーザ名とパスワードに`postgres'を使用してlocalhostに接続します
* `kubernetes` という名前のデータベースを作成します

{{% /tab %}}
{{% tab "MySQL" %}}

最も一般的な形式では、MySQLの `datastore-endpoint` パラメータは次の形式です:

`mysql://username:password@tcp(hostname:3306)/database-name`

より高度な設定パラメータを使用できます。詳細については、https://github.com/go-sql-driver/mysql#dsn-data-source-name を参照してください。

K3sの[既知の問題](https://github.com/rancher/k3s/issues/1093) のため、`tls` パラメータを指定できません。TLS通信はサポートされますが、たとえば、このパラメータに "skip-verify" に設定して、K3sで証明書の検証をスキップすることができません。

指定したデータベース名が存在しない場合、サーバーはそのデータベース名を作成しようとします。

エンドポイントとして`mysql://`のみを指定すると、K3sは以下のことを実行しようとします:

* MySQLソケット `/var/run/mysqld/mysqld.sock` に接続します。`root` ユーザーを使用し、パスワードを使用しません
* `kubernetes` という名前のデータベースを作成します


{{% /tab %}}
{{% tab "etcd" %}}

最も一般的な形式では、etcdの `datastore-endpoint` パラメータの形式は次のとおりです:

`https://etcd-host-1:2379,https://etcd-host-2:2379,https://etcd-host-3:2379`

上記は、典型的な3ノードのetcdクラスタを想定しています。パラメータには、1つ以上のカンマ区切りのetcd URLを指定できます。

{{% /tab %}}
{{% /tabs %}}

<br/>上記のことから、以下のようなコマンドを使用するとk3sという名前のPostgresSQLデータベースに接続するサーバインスタンスを起動することができます:
```
K3S_DATASTORE_ENDPOINT='postgres://username:password@hostname:5432/k3s' k3s server
```

また、次の例はクライアント証明書を使用してMySQLデータベースに接続します。
```
K3S_DATASTORE_ENDPOINT='mysql://username:password@tcp(hostname:3306)/k3s' \
K3S_DATASTORE_CERTFILE='/path/to/client.crt' \
K3S_DATASTORE_KEYFILE='/path/to/client.key' \
k3s server
```

### HA向け組み込みDQLite(試験実装)
K3sで利用されるDQLiteの使い方はSQLiteと似ています。操作方法とセットアップは簡単です。そのため、このオプションを使用するための外部設定や追加手順は必要ありません。このオプションを使用してを実行する方法については、[(試験的)組み込みDBによる高可用性]({{< baseurl >}}/k3s/latest/en/installation/ha-embedded/) を参照してください。
