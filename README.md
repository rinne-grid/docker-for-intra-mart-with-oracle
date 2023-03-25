# docker for intra-mart with Oracle Database

- intra-mart の検証環境を Docker コンテナ上に作成し、環境構築にかかる時間を削減

## バージョンについて

- AP（Resin）のコンテナについては、Git タグで管理しています。
- 利用したいバージョンのタグ指定の上 clone を実施してください。

```sh
$ git clone https://github.com/rinne-grid/docker-for-intra-mart-with-oracle im -b <tag_name>
```

| tag                              | OS                  | Oracle バージョン      | Java バージョン | 確認した Resin バージョン       | 確認した iAP バージョン    |
| -------------------------------- | ------------------- | ---------------------- | --------------- | ------------------------------- | -------------------------- |
| v1.0-oracle12c-openjdk8          | centos:7.5.1804(※1) | Oracle Database 12.2.0 | OpenJDK8        | 4.0.56 より前のバージョンで確認 | iAP 2019 Summer で確認(※1) |
| v2.0-oracle19c-openjdk11-rhel8.x | RHEL8.x             | Oracle Database 19.3.0 | OpenJDK11       | 4.0.66 で確認                   | iAP 2022 Spring で確認     |

## Docker で作成する intra-mart のシステム構成

### Docker 上の環境

| ホスト名 | コンテナイメージ                         | 目的                              |
| -------- | ---------------------------------------- | --------------------------------- |
| ap       | CentOS 7.5.1804(※1）(library/centos)     | resin-pro の実行及び war デプロイ |
| db       | Oracle Database 12.2.0.1.0(ビルドで作成) | intra-mart に関するデータの保存   |

（※1 厳密には、NTT データイントラマート社は intra-mart が動作する Linux 環境として、Red Hat Enterprise Linux 6.x、7.x のみを動作保証しているため、
本 Docker 関連ファイルを利用する場合は、あくまでも動作検証用の環境に留めておくことをおすすめします。本記事の内容によって発生した障害等について、一切責任を負いません）

## 事象別のコマンドリファレンス

| 事象                                                                                                                                    | コマンド                                                                     |
| --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Docker コンテナを開始したい                                                                                                             | docker-compose up                                                            |
| Docker コンテナをビルドしたい                                                                                                           | docker-compose build --no-cache                                              |
| Docker コンテナを終了させたい                                                                                                           | docker-compose down                                                          |
| DB データやストレージを削除して、<br>新しくテナント環境セットアップから始めたい（永続化しているコンテナのデータが全部消えるので要注意） | docker-compose up <br>docker-compose down -v<br>docker-compose up            |
| war ファイルをアンデプロイしたい                                                                                                        | docker-compose exec ap /ap-server/bin/resinctl undeploy imart                |
| war ファイルをデプロイしたい                                                                                                            | docker-compose exec ap /ap-server/bin/resinctl deploy /war/imart.war         |
| Oracle イメージを単独で作成したい場合                                                                                                   | docker build -t oracle12c_local:latest ./db/build --build-arg DB_EDITION=SE2 |

## コンテナごとの接続情報

- AP サーバー（CentOS）

| コンテナ名 | ホスト名 | ポート番号(ホスト) | ポート番号(コンテナ) |
| ---------- | -------- | ------------------ | -------------------- |
| im_ap_n    | ap       | 8888               | 8080                 |

- Oracle Database

| コンテナ名 | ホスト名 | DB 名 | ユーザ名 | パスワード | ポート番号(ホスト) | ポート番号(コンテナ) |
| ---------- | -------- | ----- | -------- | ---------- | ------------------ | -------------------- |
| im_db_n    | db       | IMART | IMART    | IMART      | 15210              | 1521                 |

- Adminer

| コンテナ名 | ホスト名 | ポート番号(ホスト) | ポート番号(ゲスト) |
| ---------- | -------- | ------------------ | ------------------ |
| im_ap_n    | adminer  | 8889               | 8080               |

## 利用手順

本 Docker プロジェクトを利用して、intra-mart の環境を構築する手順を記述します。
（この手順は、Windows10 + Docker Desktop を前提にしています。必要に応じて適宜読み替えをお願いします。）

### [1] Docker Desktop のインストール

- 下記の URL より、Docker Desktop をダウンロードし、インストールします

https://www.docker.com/products/docker-desktop/

### [2] Git のインストール

- 下記の URL より、Git for Windows をダウンロードし、インストールします

  https://gitforwindows.org/

### [3] Docker プロジェクトのダウンロードと初期設定

- 任意のフォルダで、以下のコマンドを実行し、docker プロジェクトをダウンロードします

```sh
> git clone https://github.com/rinne-grid/docker-for-intra-mart-oracle im
> cd im
```

- war ファイルの配置用フォルダを作成します

```sh
> mkdir .\ap\war
```

### [4] Oracle Database 12.2.0.1.0 をダウンロードします

- 下記の URL にアクセスし、(12.2.0.1.0) - Standard Edition 2 and Enterprise Edition を見つけ、Linux x86-64 をダウンロードします

  https://www.oracle.com/technetwork/jp/database/enterprise-edition/downloads/index.html

- ダウンロードしたファイル(linuxx64-12201_database.zip)を im/db/build フォルダに配置します

### [5] Juggling で war ファイルを作成

プロジェクト名を imart にして、必要なモジュールを選択し、設定を行います。
今回の Docker 環境をそのまま利用するためには、下記ファイルの設定を変更する必要があります

- storage-config.xml を設定する
- resin-web.xml を設定する
- 出力する war ファイル名を imart.war とする

（Docker プロジェクトの ap/.env ファイルを変更することで、別の値を指定することも可能です）

#### storage-config.xml の設定

imart/config/storage-config.xml の 19 行目付近を以下のとおりに変更します

```xml
<root-path-name>/im-data/storage</root-path-name>
```

#### resin-web.xml の設定

imart/resin-web.xml 内容を下記のとおりにします

```xml
<web-app xmlns="http://caucho.com/ns/resin" xmlns:resin="urn:java:com.caucho.resin">
    <character-encoding>UTF-8</character-encoding>

    <log-handler name="" class="jp.co.intra_mart.common.platform.log.handler.JDKLoggingOverIntramartLoggerHandler"/>
    <logger name="debug.com.sun.portal" level="warning" />

	<!-- im_service(im_asynchronous) -->
	<resource jndi-name="jca/work" type="jp.co.intra_mart.system.asynchronous.impl.executor.work.resin.ResinResourceAdapter" />
	<jsp>
		<recycle-tags>false</recycle-tags>
	</jsp>
	<database jndi-name="jdbc/default">
		<driver>
		   <type>oracle.jdbc.driver.OracleDriver</type>
		   <url>jdbc:oracle:thin:@db:1521/RNGD</url>
			<user>IMART</user>
			<password>IMART</password>
		</driver>
		<max-connections>20</max-connections>
		<prepared-statement-cache-size>0</prepared-statement-cache-size>
	</database>
	<session-config>
	    <reuse-session-id>false</reuse-session-id>
		<session-timeout>30</session-timeout>
	</session-config>

	<mime-mapping extension=".json" mime-type="application/json"/>
</web-app>
```

#### war ファイルの出力

imart.war という名称で war ファイルを出力したら、
プロジェクトの im/ap/war フォルダの中に、war ファイルをコピーします

### [6] intra-mart のサイトから Linux の resin-pro をダウンロード

- intr-mart のサイトにアクセスし、プロダクトファイルダウンロードボタンを押下します。

  https://www.intra-mart.jp/download/library/

ライセンスキーを入力すると、ダウンロード可能なファイル一覧が表示されます。

なお、intra-mart サイトにも書いているとおり、.tar.gz が Linux 用の resin-pro になります。

https://www.intra-mart.jp/download/product/iap/setup/iap_setup_guide/texts/install/linux/resin_linux.html

最新の Resin<b>resin-pro-4.0.xx.tar.gz</b>を入手します。

### [7] Docker プロジェクトのフォルダに resin-pro をコピー

- 上記の resin-pro.4.0.xx を展開し「resin-pro.4.0.xx」の名称を resin-pro に変更します
- resin-pro フォルダを im/ap フォルダにコピーします

### [8] JDBC ドライバー(ojdbc8.jar)をダウンロードし、im/ap/resin-pro/lib にコピー

- 下記の URL にアクセスし、ojdbc8.jar を見つけ、ダウンロードします

  https://www.oracle.com/technetwork/database/features/jdbc/jdbc-ucp-122-3110062.html

- ojdbc8.jar を im/ap/resin-pro/lib にコピーします

### [9] シェルスクリプトの改行コードの確認とプロジェクトのフォルダ構成の確認

- 【！】重要【！】db/build/scripts_setup/01_create_imart.sh の改行コードが LF になっていることを確認します
- フォルダを確認し、以下の構成と同じになっていることを確認します
- ポイント
  - im/ap/resin-pro フォルダがあり、直下に automake や lib フォルダ等が存在する
  - im/ap/resin-pro/lib フォルダ内に odbc8.jar ファイルが存在する
  - im/ap/war フォルダがあり、imart.war ファイルが存在する
  - im/db/build フォルダ内に linuxx64_12201_database.zip が存在する

```
im
├─ap
│  │
│  ├─resin-pro
│  │  ├─lib
│  │  │   ├─ojdbc8.jar
│  │  │   │
│  │  │   ├─activation.jar など
│  │  │
│  │  ├─automake  など
│  │
│  ├─war
│  │  └─imart.war
│  │
│  └─Dockerfile
│
├─db
│  ├─build
│  │  ├─linuxx64_12201_database.zip
│  │  │
│  │  ├─checkDBStatus.sh など
│  │
│  ├─scripts_setup
│  │  └─01_create_imart.sh
│  │
│  └─scripts_startup
│
├─.env
├─.gitignore
├─docker-compose.yml
└─README.md

```

### [10] 必要に応じて、設定ファイルを変更する

- im/ap/resin-pro/conf/resin.properties の 82 行目付近 - jvm_args

-Xmx, -Xms の値が、初期状態だと 8192m(8GB)が設定されているため、自分の PC のメモリ状況に合わせて変更します

```ini
jvm_args : -Dfile.encoding=UTF-8 -Djava.io.tmpdir=tmp -Xmx4096m -Xms4096m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=30 -XX:NewSize=512m -XX:MaxNewSize=512m -XX:+CMSClassUnloadingEnabled -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+HeapDumpOnOutOfMemoryError -Xloggc:log/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=10M
```

- HTTP プロキシの設定 im/.env

社内ネットワーク等で、プロキシサーバーを経由する必要がある場合、.env の HTTP_PROXY、HTTPS_PROXY に値を設定します

```env
HTTP_PROXY=http://user:password@server:port/
HTTPS_PROXY=http://user:password@server:port/
```

### [11] Docker コンテナの起動

- プロジェクトフォルダに移動します

```sh
> cd any_folder\im
```

- docker-compose を利用し、コンテナを起動します

```sh
> $ docker-compose up -d ; docker-compose logs -f
```

イメージ作成の際にアップデート等の処理が走るので、15 分程度～ 30 分程度かかります。

terminal にいろいろと出力されていきます。
以下の表示が出たら終了と判断して良いと思います。Ctrl+C 等でを強制終了します。

```
db_1       | #################
db_1       | DATABASE IS READY TO USE!
db_1       | #################
db_1       | The following output is now a tail of the alert.log:
db_1       | EXTENT MANAGEMENT LOCAL AUTOALLOCATE
db_1       | SEGMENT SPACE MANAGEMENT AUTO
db_1       | 2019-03-07T14:11:29.343682+00:00
db_1       | RNGD(3):Completed: CREATE TABLESPACE IMART
db_1       | DATAFILE '/opt/oracle/oradata/ORCL/XXXX/IMART.dbf' SIZE 7000M REUSE
db_1       | LOGGING
db_1       | ONLINE
db_1       | BLOCKSIZE 8K
db_1       | EXTENT MANAGEMENT LOCAL AUTOALLOCATE
```

ホストマシン側から Oracle Database に接続したい場合は下記の指定を行います。

(設定を変更していない場合)

| ユーザ ID | パスワード | SID                  |
| --------- | ---------- | -------------------- |
| IMART     | IMART      | localhost:15210/RNGD |

（Docker for Windows や Docker for Mac の場合は、上記 IP アドレスではなく localhost:15210 もしくは自分で設定しているホスト名や IP アドレスを指定します。）

- resin の index ページに接続します

http://localhost:8888

![docker-toolbelt](http://www.rinsymbol.sakura.ne.jp/github_images/docker/resin-top.PNG)

- resin のページが開けることが確認できたら、war ファイルをデプロイします

```sh
> docker-compose exec ap /ap-server/bin/resinctl deploy /war/imart.war
```

コンテナ内の resin-pro の場所は、/ap-server です。
im/.env ファイル内の変数を変更することで、お好きな場所を指定できます。

- デプロイのコマンドが終了して、数分経ったら intra-mart のセットアップページにアクセスします
  - 503 Service Temporarily Unavailable が発生する場合は、もう少しだけ待ってあげてください。

http://localhost:8888/imart/system/login

- 無事にテナント設定画面が表示されるので、テナント環境セットアップを実行します

![tenant1](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant.PNG)

- テナント ID は imart を指定します

![tenant2](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant2.PNG)

- リソース参照名は一覧に表示されたものを選択します

![tenant3](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant3.PNG)

- テナント登録を行い、しばらく待ちます

![tenant4](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant4.PNG)

- テナント環境セットアップが適切に動作しているかについて、ログを確認したい場合以下のコマンドを実行します

```
$ docker-compose exec ap bash
$ less -f ./log/jvm-app-0.log
```

- データベースやストレージ情報は Docker Volume に保存しているため、データは永続化されています
  - 一度、docker-compose down で終了し、もう一度 docker-compose up を試して、システムログイン画面にアクセスすると、ダッシュボードが表示されることがわかります

![dashboard](http://www.rinsymbol.sakura.ne.jp/github_images/docker/dashboard.PNG)
