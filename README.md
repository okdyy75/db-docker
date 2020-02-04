# db-docker
各DB環境の検証用Docker

## 概要
ユーザーは`local`、DBは`DB`として作成  
文字コードはUTF-8、タイムゾーンはミドルウェアによるのでDockerより指定  
DBビルド時に「docker-entrypoint-initdb.d」に実行SQLを配置する

## 最低要件

### MacにDockerとDocker-composeをインストール

公式からDockerをワンライナーでインストール  
https://github.com/docker/docker-install

公式からDocker Composeのインストール  
最新版をインストール  
http://docs.docker.jp/compose/install.html


## My SQL
version 5.7

sql-modeのデフォルトは下記の通り
```
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

### 接続手順
```
dc exec mysql bash
mysql> mysql -ulocal -plocal db

```

## Oracle
version 19.3.0-ee

こちらを参考にさせて頂きました  
DockerでOracle Databaseを使う  
https://qiita.com/gorilla0513/items/f22e8cce4e08da031abe

### Oracle DBのDockerイメージ作成
オラクルのコンテナを起動する前にDockerイメージを作成する。手順は下記の通り

Oracleの公式Docker Image  
https://github.com/oracle/docker-images  

Oracle Databaseの公式Docker Image  
https://github.com/oracle/docker-images/blob/master/OracleDatabase/SingleInstance/README.md  

DokcerにOracle Databaseの公式イメージがないので、Oracleが用意したimageをローカルで作成する
```
git clone https://github.com/oracle/docker-images.git
```

Oracle本体をDL、指定のフォルダに移動コピー（オラクルのアカウントを作成する必要あり）
```
cp ~/Downloads/LINUX.X64_193000_db_home.zip docker-images/OracleDatabase/SingleInstance/dockerfiles/19.3.0/
```

ビルドシェルを実行し、Docerイメージを作成
```
cd ./docker-images/OracleDatabase/SingleInstance/dockerfiles
sh buildDockerImage.sh -v 19.3.0 -e
```

OracleのDBに接続する
```
sqlplus sys/<your password>@//localhost:1521/<your SID> as sysdba
sqlplus system/<your password>@//localhost:1521/<your SID>
sqlplus pdbadmin/<your password>@//localhost:1521/<Your PDB name>
```

### 接続手順
```
dc exec oracle bash
oracle> sqlplus pdbadmin/SYS@//localhost:1521/PDB
```

- Oracleは他のDBに比べて概念が難しいため初期設定をどこまでするか難しい。。。
- テーブルを作成するにはまずユーザーが必要
MySQLで言うところのDBがOracleで言うユーザー（スキーマ）に近いと思われる
- TABLE SPACE、TEMPORARY TABLESPACEについてはこちらが参考になりそう  
[【ORACLEのお勉強#01】テーブル、表領域、データファイル、ディスクの関係](https://el.jibun.atmarkit.co.jp/yuu1/2017/08/oracle_oracle_redoundo_oracle_10oracle.html)  
- Oracleの権限についてはオブジェクト権限、システム権限があり、システム権限が与えられないとユーザーを作成してもテーブル作成できない。ひとまず定義済みDBAロールを付与。

## PostgreSQL
version 12.1

ロケールはDB作成時にしか設定できないので、Dockerfile起動時に設定

### 接続手順
```
dc exec postgres bash
postgres> psql -U local -d db

```

## Dockerコンテナ起動
```
dc down
dc build --no-cache
dc up -d
```
