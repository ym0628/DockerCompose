# DockerComposeでPHP・MySQLの開発環境を構築する

いまやチーム開発には不可欠となりつつDocker、特にDockerComposeについての環境構築を学んでいき、その備忘録をこちらの残していきます。


## <font color="Salmon">参考サイト</font>


https://www.youtube.com/watch?v=JP2f1R432Fw

上記はDockerの基礎として把握しておく。
実践は以下の動画「DockerCompose」の環境構築の方法が分かりやすいだろう。

https://www.youtube.com/watch?v=cpoempEOtow

こちらの動画をメインに学習していきます。


## <font color="Salmon">事前情報の確認</font>

### DockerとDockerComposeの違い

- DockerComposeはDockerの機能をさらに便利にしたDockerの上位互換みたいなもの。
- Dockerfileから生成するイメージやコンテナを、複数同時に作ったり、操作することができる。
- 無印のDockerだと、 Dockerfileからイメージをビルド、Dockerfileからコンテナをビルド、、、といった具合に、イメージ一つ一つに対してDockerfileを作る必要があった。
- それを解消し、より簡単にイメージ・コンテナの作成を複数同時に作成できるようにし、チーム開発の効率性を高める事ができるようになったのが、DockerComposeという事だ。

### DockerComposeの利用方法
- MacでDockerComposeを使用するにはベースとなるDocker desktop for Macをインストールするところから始まる。


### 疑問点をまとめる
- phpはMacには標準で入っていると聞くが、どうやって確認するの？
- →→入ってなかった。
- →→たぶん、Dockerfileに`FROM php.... apache...`みたいに定義する事で、Dockerの中に指定したバージョンのphpと Apacheを導入したイメージが作成できるみたいだ❗️
- そのイメージを元に`run`コマンドでコンテナを作ればオーケーっぽい❓❗️
- Git管理したり、Laravelを導入したりなど、Dockerの環境構築との手順は、どっちが先になるの？
- →→まだ分からない🤷



## <font color="Salmon">実装・設定の記録</font>


こちらに個人の実装の記録やメモを残していきます。

<br>

### Apacheのバージョンを確認
- ルートディレクトリで下記コマンドを実行

```terminal
% apachectl -v

Server version: Apache/2.4.51 (Unix)
Server built:   Feb 12 2022 02:40:22
```
- Apacheはインストール済みだと分かりました。
- そもそもMacはプリインストールされているっぽい？

### PHPのバージョンを確認
- 以下コマンドをルートディレクトリで実行します

```terminal
% php -v

=> zsh: command not found: php
```

- phpはインストールされていないっぽいです。
- プリインストールされていると思っていたのですが、されていないっぽい。
- 自身のPC環境を確認します。

```terminal
MacBook Air （M1, 2020）
Apple M1
macOS Monterey
shell: zsh
```

### Homebrewでphpのインストール＋実行環境のセットアップをする手順について

- こちらの記事を参考に、phpをインストールする法法があるようです。
- https://zenn.dev/ryuu/articles/setup-php-brew

https://zenn.dev/ryuu/articles/setup-php-brew

- これはこういう方法で、一つの選択肢として一旦、実行は保留します。
- DockerCompose内でのみ、phpをインストールし、実際にコンテナ内でのみphpが使える環境になるのかを確かめるという意味でも、まずはDockerでの環境構築を目指していきます。


```terminal
# インストール可能なphpバージョンを確認
$ brew search php

Warning: Error searching on GitHub: GitHub API Error: Requires authentication
```
- んん、、なんかダメそうな感じ。


```terminal
# ルートディレクトリで下記コマンドを実行
（※バージョン指定でPHP8.0をインストールはせず、バージョンは指定せずに実行してみます）
$ brew install php
```

- インストールしたPHPは自動的にパスが通らないため、シェルのプロファイルから`パスを手動で通す`必要があるらしい。
- `zshシェル`を使っている場合は、以下のコマンドをターミナルで実行し、パスを通す。

```terminal
$ echo 'export PATH="/opt/homebrew/opt/php@8.0/bin:$PATH"' >> ~/.zshrc
$ echo 'export PATH="/opt/homebrew/opt/php@8.0/sbin:$PATH"' >> ~/.zshrc
```


- コマンドを実行後、パスを読み込ませるためにシェルを再起動します。
- その後、phpのバージョン確認コマンドを叩いて表示されればOK。

```terminal
$ シェルを再起動する
$ php --version
```

- 続いて、シェルのプロファイル内に次の２行を追記します。
- シェルを再読み込みし有効化しておきます。

```terminal:~/.zshrc
$ export LDFLAGS="-L/opt/homebrew/opt/php@8.0/lib"
$ export CPPFLAGS="-I/opt/homebrew/opt/php@8.0/include"
$ シェルを再読み込みし有効化する。
```

- ここまでが、`Homebrewでphpのインストール＋実行環境のセットアップをする手順`である。


### 【実践】DockerComposeでPHP・MySQLの開発環境を構築

- ある程度、予備知識がついたと思うので、早速DockerComposeでのphp環境構築を実行に移していきます。
- まずは動画を参考に、手順をざっくり確認していきます。


<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/762f4f2e-3214-fbeb-4fe2-602ec983f168.jpeg" alt="Dockerの構築フローのイメージ" width=50% height=50%>


<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/966c12b3-0a83-ddda-1019-3ba00b803ca9.jpeg" alt="" width=50% height=50%>


<br>

### <font color="Green">0. Gitでバージョン管理する事前設定</font>

- まずはGit管理下におきたいので、GitHubで`DockerCompose`リポジトリを作成し、`workspace`ディレクトリ配下に`git clone`します。

```terminal
バージョン管理するためにGitHubにprivateリポジトリを作成する。
$ GitHubにログインしてnewページから新しいリポジトリを作成
$ README.mdはあらかじめリモートリポジトリで自動生成しておいた。これでリモートリポジトリで、自動的にmainブランチとREADME.mdが生成される。


ローカルにて、クローンしたいディレクトリにcdコマンドで移動してクローンする。
$ git cd ~/workspace
$ git clone git@github.com:******/DockerCompose.git

ローカルにて、クローンしたディレクトリに移動する。
$ cd DockerCompose
$ git branch  # 予めREADME.mdがmainブランチに作成されている状態になっている。


readmeブランチにて、README.mdファイルを適当に更新してみる。
$ git checkout -b first_commit
$ git branch  # ブランチが移動している事を確認
$ ローカルでREADME.mdファイルを編集・更新

一度コミット〜プルリクエスト〜マージ〜プルできるか確認。
$ git add .
$ git commit -m "first_commit"
$ git log
$ git push
$ git push --set-upstream origin first_commit
$ git log
$ プルリクエストを作成
$ リモートでマージ
$ git checkout main
$ git pull origin main

ローカルのブランチはこまめに削除してOK。
$ git branch -d first_commit
```


- ここまでに、Git管理に成功し、ローカルにリポジトリができていればOK。


<br>

### <font color="Green">1. Dockerアプリのインストール</font>

- すでにインストール済みなので飛ばします。

<br>



### <font color="Green">2. 必要なフォルダ・ファイルを作成</font>

- 以下のディレクトリを作っていきます。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/c42a014e-99f2-e1c8-9566-26d1cac2d8e2.jpeg" alt="ディレクトリ構成のイメージ" width=50% height=50%>


```terminal
$ DockerCompose % tree
.
├── README.md
├── app
│   ├── Dockerfile
│   └── src
│       └── index.php
├── compose.yml
└── mysql
    └── initdb.d
        └── init.sql

5 directories, 5 files
```

```terminal
$ git status -u
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	app/Dockerfile
	app/src/index.php
	compose.yml
	mysql/initdb.d/init.sql
```


<br>
<hr>


***一旦ここまでをコミットするのですが、その前にGit管理をしやすくするための設定を行います***

- `.gitignore`ファイルをルートディレクトリ配下に作成し、`.DS_Store`をGit管理から除外します。
- さらに空のフォルダがコミット対象から外れてしまうリスクを回避するため、各ディレクトリ配下に`.gitkeep`ファイルを作成します。

```terminal
# Mac OS
.DS_Store
```

```terminal
$ touch .gitkeep # 各ディレクトリに作成
```

```terminal
$ git status -u

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
	.gitkeep
	app/.gitkeep
	app/Dockerfile
	app/src/index.php
	compose.yml
	mysql/.gitkeep
	mysql/initdb.d/init.sql
```

***ここまで出来たらコミット〜プルリクエスト〜マージ〜プルまで実施します***

```terminal
$ git add .
$ git status -u
$ git commit -m "【Add】DockerおよびPHP環境構築に必要なフォルダ・ファイルを作成"
$ git log
$ git push
$ プルリクエストを作成
$ リモートブランチmainにマージ
$ git checkout main
$ git pull origin main
$ git log
$ git checkout log # 再びローカルの実装ブランチに戻って次の作業に備えます
```

- さらに、ここまでの記録をREADME.mdに追記して更新します。
- 更新したら`README.mdの更新2`のコミットも行っておきます。



<br>
<hr>


### <font color="Green">3. Dockerfileの作成</font>

***<font color="Red">※動画10分00秒あたりから</font>***


- `最終目標`：DockerComposeでコンテナを複数個を一発で簡単に作成・削除できるようにしたい。
- そのためには、`コンテナ`を作るための`イメージ`が必要。
- `イメージ`を作るには`Dockerfile`を自作していく必要がある。
- ということで、次は`Dockerfile`の作成を行なっていきます。
- `Dockerfile`に以下のように記述していきます。

<br>

- `php`用の`Dockerfile`を作るためのイメージは`Docker Hub`という公式ドキュメントから参照します。
- https://hub.docker.com/
- dockerhubのサイトから検索で`php`でキーワード検索する
- すると、`DOCKER OFFICIAL IMAGE`というphpのイメージがすぐ見つかるのでそれをクリック。
- `How to use this image`でイメージを指定行きましょう。
- 今回は、`php`と`Apache`を導入したいと言う目的があるので、それを検索して調べる。
- 手順としてはphpでオフィシャルイメージのページから`Tags`タブをクリックし、検索窓に`apache`と入力して検索する。
- すると、phpとapache、両方を含んだイメージの使用例、ドキュメントが見つけられます。
- 今回は、このイメージで試してみます。
- <a href="https://hub.docker.com/layers/library/php/8.3.0-apache/images/sha256-10d28c0b61b45dc045ec7a2d0d853e90d275c14d4f2e756e126e8d0bd8457a92?context=explore">php:8.3.0-apache linux/arm64/v8</a>
- 理由は、Applesillicon、M1チップで使えるイメージが、これだと考えたから。参考にしたのはこちらのドキュメントです。
- https://matsuand.github.io/docs.docker.jp.onthefly/desktop/mac/apple-silicon/
- phpのバージョンはひとまず最新の8.3を採用してみます。



```dockerfile
# ~/workspace/DockerCompose/app/Dockerfile
# phpとapacheのイメージ
FROM php:8.3.0-apache
```

- イメージはこれで決まり。
- 次に MySQLを導入するための記述をしていく。
- 今回はphpと Apacheによって構成されたWebサーバーから、DBすなわちMySQLへ接続する必要がある。
- MySQLを指定するには、パッケージというものをインストールする必要がある。
- インストールするには同じく`Dockerfile`内に、`RUN`というコマンドを打ち込む必要がある。
- 以下のように記述します。


```dockerfile
# ~/workspace/DockerCompose/app/Dockerfile
# MySQLのパッケージ
RUN apt update \
    && docker-php-ext-install pdo_mysql
```

***[意味]***
- `apt update`しつつ
- `&&` かつ次のコマンドを実行します。
- `docker-php-ext-install`でphpの拡張ファイルをいい感じにインストールしてね。
- `pdo_mysql`でMySQLに接続してね。

これらの指令を行うコードとなる。

<br>

- 続いて、メインコンテンツとなる`index.php`の中身を実装していく。
- 今回のコードは主題ではないので、多くは語られていない。
- 大まかな内容としては、
- インストールしたMySQLパッケージから特定のクエリを発行し、`var_dump`で出力するという、非常に簡素なビューおよびモデル（DB）の連携を示しています。
- ソースコードは動画の提供者様のGitHubから拝借します。
- https://github.com/fuku-youtube/php-mysql-docker-compose



```php
<!-- ~/workspace/DockerCompose/app/src/index.php -->

<?php

// 接続
// hostはコンテナ名を記載する
$dsn = 'mysql:dbname=test_db;host=run-php-db;';
$user = 'test';
$password = 'test';

try {
    $pdo = new PDO($dsn, $user, $password);
    $sth = $pdo->query("SELECT * FROM users WHERE id = 1");
    $user = $sth->fetch(PDO::FETCH_ASSOC);
    var_dump($user);
} catch (PDOException $e) {
    print('Error:'.$e->getMessage());
    exit;
}
```

- 本工程では、以下の通り、2つのファイルにコードを記述しました。

```terminal
$ git status -u
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   app/Dockerfile
	modified:   app/src/index.php
```

***<font color="Red">※動画14分23秒あたりから</font>***

***MySQLの準備***

- 作成したDockerfileからイメージを作成していく。
- その前に、MySQLファイルの定義が先。

```sql
CREATE TABLE test_db.users (
    id      INT             NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    age         INT,  
    PRIMARY KEY (id)
);

INSERT INTO `users` VALUES (1, 'Fuku', 30)
```

- `test_db`というタイトルのデータベースに、`test_db.users`という名前のテーブルを`CREATE`してね。という内容。
- `test_db`というタイトル名は、先ほどの`index.php`にて定義している。
- `users`テーブルから情報をひとつ取得すると言うクエリの定義も、先ほどの`index.php`にて定義している。
- `users`には、`id`や`first_name`そして`age`カラムを用意している。
- ここまで定義したテーブルやテーブル内のカラムに対して、、、
- 次の行の`INSERT INTO`にて、引数に id=1, first_name=tanaka, age=35という情報を入力してユーザーを一つ作成するようにこの`SQL`ファイルで指示している。
- ざっくり言うと、ここではテーブルを作って、その中にデータを入れるというクエリを、この`init.sql`ファイルでは定義した。

<br>

### <font color="Green">4. docker-compose.ymlの実装</font>

***<font color="Red">※動画15分11秒あたりから</font>***

- `compose.yml`ファイルにコードを記述していきます。
- これは今回の主題である`DockerCompose`というツールを使って開発環境を構築するための指示書みたいなファイルとなります。
- ここで、使用するDBやWebサーバーを指定していったり、コンテナ起動のポート番号の指定、コンテナ削除後の永続化を実現する定義などを記述します。
- ソースコードは動画up主様のGitHubをそのまま転載させていただきました。
- https://github.com/fuku-youtube/php-mysql-docker-compose

```yml
# パス：　~/workspace/DockerCompose/compose.yml
# 1番上の階層、決まり文句のようなもの。
services:
  # 2番目の階層、作りたいコンテナを定義する。任意の名前をつけられる。
  php-app:
    # コンテナ名
    container_name: run-php-app
    # ビルドするDockerfileの場所（相対パスで指定してあげる。）
    build: ./app
    # ポート（左側はローカルPCのポート番号:右側はコンテナ側のポート番号）
    # ポート番号も任意で決めることはできるが、番号にはルールがあるのでカスタムする際は知識が必要。
    ports:
      - "18080:80"
    # ローカルPCとコンテナ間でディレクトリをバインド（同期）できる
    # （左側はローカルPCの相対パス:右側はコンテナのパス）を指定する
    # 「/var/www/html/」は Apacheのドキュメントルートがこれであるというルールに従っている。
    volumes:
      - ./app/src:/var/www/html/
    # 利用するネットワーク
    networks:
      - php-mysql-networks
    # 指定したサービスの後にコンテナを起動する
    # 必ずDBを先に起動してからコンテナを起動するように指示している。
    depends_on:
      - php-db
  php-db:
    # 今回はDockerfileを使うまでもない簡素なappであるため、省略している。いきなりイメージの指定から記述している。
    # imageはDockerHubの公式ドキュメントに記述の仕方が指南されているのでチェックする。
    image: mysql:8.0
    # コンテナ名「run-php-db」は、index.phpの「host=run-php-db」で指定した名前を持ってきている。
    # ここの名前を間違えると動かないので注意する。
    container_name: run-php-db
    ports:
      - "3307:3306"
    # コンテナの環境変数を指定できる
    # コンテナ内で共通に使えるようになる設定・定義であると認識しておく。
    environment:
      - MYSQL_ROOT_PASSWORD=root # MySQLに接続するスーパーユーザー最高権限を持つユーザーのパスワード
      - MYSQL_USER=test # MySQLを操作するユーザーの名前
      - MYSQL_PASSWORD=test # MySQLを操作するユーザーのパスワード
      - MYSQL_DATABASE=test_db # これを指定してあげるとDBを勝手に作ってくれる。この名称はindex.phpで定義したDB名を指します。
    # volumesは、ローカルで設定したinit.sqlの設定をコンテナで実行したいよね。という希望を叶える定義の場所
    # DBの永続化のボリュームも指定できる
    volumes:
      # 左はローカルPCのMySQLの相対パス：右はコンテナのMySQL実行場所の相対パス
      # /docker-entrypoint-initdb.dは、MySQLを勝手に実行してくれる、Dockerで決められている便利な場所である。
      - ./mysql/initdb.d:/docker-entrypoint-initdb.d
      # DBの永続化をさせる定義。コンテナを削除したときに、DBの中身も消えてしまう。それを防ぐのが下記の設定である。
      - mysql-data:/var/lib/mysql
    networks:
      # phpとDBで同じネットワーク名を定義することで、コンテナ同士で接続してくれるようになる。
      # あえてここで任意の名前を決めなくてもいいっちゃいいのだが、本動画では分かりやすく伝えるために任意の名前をphpとdbで揃えている。
      - php-mysql-networks
  # DBをUIで確認できるツール
  php-adminer:
    container_name: adminer
    # イメージ名もDockerHubで探すことができる。
    image: adminer:4.8.1
    ports:
      - 8081:8080
    networks:
      # ここも、phpとdb同様、同じ名前のネットワーク名を指定してあげる。
      - php-mysql-networks
      # ここも同様、同じ名前のDB名を指定してあげる。
    depends_on:
      - php-db
# 任意のボリュームを作る(DBの永続化用)
# serviciesと同じく1階層目に書くのがルール。
volumes:
  # ここでmysql-dataという名前のDBを指定（⤴のvoluemsの永続化設定で指定した名前）することで、「php-db」のDBのデータを永続化してくれる。
  mysql-data:
# 任意のネットワークを作る(わかりやすくするため)
# ここも第一階層に作らなければならない。
networks:
  php-mysql-networks:
```

***上記コードのメモ***

- `compose.yml`の記述方法にはルールがある。
- ルールについては公式リファレンスなどを参考にする。
- Google検索で「compose yml 公式リファレンス」などと検索してみて調べる。
- https://docs.docker.jp/v1.12/compose/compose-file.html

<br>
- 1階層目は決まり文句`sevices`を配置
- 2階層目は、作りたいコンテナを定義する。名前は任意でOK。ここでは、以下のように3つのコンテナを作成する定義を実装している。以下3つのコンテナ名を定義した。
- 今回は、servicesを3つ作りたい。
- `php`　と `MySQL(DB)`と `Adminer（DBをUIで確認できるDB管理ツール）`、この3つを定義してあげたわけだ。

```yml
php-app:
php-db:
php-adminer:
```

<br>


### <font color="Green">5. Dockerfileからイメージを作成・コンテナを起動</font>

- 上記の実装ができたら`DockerCompose`でのコンテナ作成の準備完了。
- ここからターミナルでコマンドを使い、Dockerfileからイメージの作成〜コンテナの起動をおこなっていくことができる。


1. まずはDockerDesktopアプリケーションを起動する。
1. ターミナルで対象のルートディレクトリに移動する。`cd ~/workspace/DockerCompose`（このとき、配下に必ず`compose.yml`ファイルがあるディレクトリでコマンドを実行する）
1. 次のコマンドを叩く `docker compose up -d`
1. 上手くいけば、5分くらい待つとイメージ・コンテナが作成・起動する

<br>

- 下記のように、コンテナ・イメージ・ボリュームがDockerDesktopに生成された。
- `in use`は`使用中`という意味
- `Running`は起動中という意味

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/f9a1d6bf-927d-1278-a91c-1f1a7cc41e35.jpeg" alt="コンテナ・イメージ作成の結果画像" width=50% height=50%>


- 次のように、DockerDesktopアプリのコンテナメニューから、phpのポスト番号をクリックすると、Apacheを解して、Webサーバーが起動し、ブラウザのローカルホストでUIを確認することができる。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/670e62d7-a72a-4973-1613-40815ee7a3cb.jpeg" alt="" width=50% height=50%>

- Adminerを使えば、データベースの中身をブラウザUIで確認することもできる。
- `compose.yml`で定義したユーザー名やパスワード、DB名を入力すればログインして中身を確認できる。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/d409a889-f902-a257-c934-d4f7b20ab765.jpeg" alt="" width=50% height=50%>


<br>

### <font color="Green">6. コンテナの停止・削除</font>

- 最後に`コンテナは使い捨て`と言うように、作業が終わったら簡単に削除することができるので、やってみる。
- ターミナルコマンドは`docker compose down`
- これでコンテナを削除することができる。
- イメージは残るので、次にコンテナを起動させる際はもっと早く起動させることができる。
- DBは永続化させる設定を行なっているので、以前作成したDBの`user`情報は残ったまま、コンテナをビルドすることができで便利。

<br>

- コンテナは削除された。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/ee3befcd-2ea9-a493-c630-59bcb17c0355.jpeg" alt="" width=50% height=50%>

- イメージ・DBは残っている。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/45ea2400-997b-49a6-d298-6bb4e25b55b4.jpeg" alt="" width=50% height=50%>

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3486945/a76fd470-a02a-ab28-173a-970780149540.jpeg" alt="" width=50% height=50%>


今後は以下コマンドでコンテナの起動・削除を行う。
```terminal
$ docker compose up -d  # または $ docker compose up
$ docker compose down
```

- 以上で動画の内容は終了となります。
- Laravelの環境構築をどうするかは、また改めて別ドキュメントを調べてやってみることにします。
- ここまでをコミットして一旦終わりとします。






<br><br><br><br><br><br><br>


## <font color="Salmon">🗒よく使うタグ</font>

`## <font color="Salmon">サーモンピンク</font>`

`### <font color="Green">グリーン</font>`

`<img src="" alt="" width=50% height=50%>`


<br><br><br><br><br><br><br>



