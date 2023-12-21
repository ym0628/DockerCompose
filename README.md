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

```terminal
$ 
```




<br>

### <font color="Green">3. Dockerfileの作成</font>





<br>

### <font color="Green">4. Dockerfileからイメージを作成</font>





<br>

### <font color="Green">5. コンテナの作成・起動・停止・削除</font>







<br><br><br><br><br><br><br>


## <font color="Salmon">🗒よく使うタグ</font>

`## <font color="Salmon">サーモンピンク</font>`

`### <font color="Green">グリーン</font>`

`<img src="" alt="" width=50% height=50%>`

<br><br><br><br><br><br><br>
