### git の仕組み

git はスナップショットを記録している。

バージョン 1:ファイル A、B、C

バージョン 2:ファイル A、B1、C1

バージョン 3:ファイル A1、B1、C2

バージョン 2 を保存する時、ファイル B1、C1 はファイル B、C との差分ではなく、
ファイル B1、C1 を丸ごと保存する。
バージョン 3 を保存する時、ファイル A1、C2 はファイル A、C1 との差分ではなく、ファイル A1、C2 を丸ごと保存する。

スナップショットとして保存することで複数人での開発(ブランチを切ったりマージする時)をスピードアップさせることができる。

### バージョン記録によってできる事

バージョン記録の際、git はコミットを行う。
それぞれのコミット は直前のコミットとして保存している為、コミットをたどることで、以前の状態に戻すことができる。

### 作業の流れ

ローカル(自分の PC)のワークツリー上での変更のスナップショットを記録して、ローカルレポジトリに保存。
ローカルレポジトリから、GitHub(リモートレポジトリ) にアップする。

ワークツリー：作業ディレクトリやフォルダ
レポジトリ：履歴データの保管場所(データベース)

リモート：自分の PC とは異なる場所、を指す。

逆に、他人の変更をローカルに取り込む場合の作業の流れは、
GitHub(リモートレポジトリ)から変更内容をローカルレポジトリに取得し、
その後、ワークツリーに内容を反映させる。

### ローカルの 3 つのエリアについて

ローカルは、ワークツリー、ステージ、(ローカル)レポジトリの 3 つのエリアに分かれている。
ワークツリー：同上
ステージ：コミットする変更を準備する場所。

> git add .

※ステージが存在する理由
全ての変更が完了していなくても途中で GitHub にアップしたい、といった場合に、
ステージに反映させたファイルだけをレポジトリにスナップショットとして記録する為。

レポジトリ：ステージに追加されたファイルをスナップショットとして記録する為の場所。

> git commit -m "..."

### Git でのデータ構造

git add を実行する際、まずワークツリー上の index.html というファイルの圧縮ファイル A がレポジトリに保存する。
その後、ステージ上に index.html と、圧縮ファイル A を紐づけるインデックス情報を保存する。

git commit を実行する際、ステージ上のインデックス情報のファイル構成を元に、ツリー情報 1 がレポジトリに作成する。
その後、

> ツリー情報 1、作成者、日付、コミットメッセージ

をひとまとめにしたコミット情報 1 をレポジトリに作成する。

git add、git commit が完了した時点で、ローカルレポジトリには圧縮ファイル A,ツリー情報 1、コミット情報 1 の 3 つのデータが保存されている。

この状態から、新たに home.css というファイルをコミットする場合、
まず、git add で圧縮ファイル B をレポジトリに追加し、ステージのインデックスに home.css と圧縮ファイル B を紐づけたインデックス情報が追加される。

git commit によって、インデックス情報を元に新たにツリー情報 2 がレポジトリに追加される。そして、

> ツリー情報 2、親コミット情報、作成者、日付、コミットメッセージ

をひとまとめにしたコミット情報 2 をレポジトリに作成する。

コミットに直前のコミット情報を保持しておくことで、変更分をたどれるようにしておく。

git add でインデックス作成、git commit でツリー作成が分かれている理由
→ インデックス作成をする度に、インデックスは上書きされるが、ツリーは別ファイルとして保存されることから、データの管理が非効率的になる為。

### 圧縮ファイルについての詳細

圧縮ファイルはレポジトリに追加したいファイルの中身そのものを圧縮したもので、正確には「blob（ブロブ）オブジェクト」と言う。

圧縮ファイルのファイル名はハッシュ ID になる。

ハッシュ ID：ヘッダー（ファイル内容の文字数など、ファイルのメタ情報）とファイル内容を、SHA-1 というハッシュ関数で 40 文字の英数字に変換したもの。ハッシュ ID のうち、先頭 2 文字をディレクトリ名に、残り 38 文字をファイル名にして保存する。

> git hash-object memo.md

25e878b283184ba9439cc402c26bbc84ec023da1

> git add memo.md

> tree .git

C:\USERS\..\DOCUMENTS\GITHUB\GITHUB_THEORY\.GIT
├─hooks
├─info
├─objects
│ ├─25
│ ├─info
│ └─pack
└─refs
├─heads
└─tags

25 フォルダの直下に e878b283184ba9439cc402c26bbc84ec023da1 ファイルが保存されている。

重要な点は、ハッシュ ID はファイルの中身が同じであれば必ず同じハッシュ ID になる為、git add しても追加で圧縮ファイルが作られるわけではないという事。

### ツリーファイルについての詳細

圧縮ファイルのファイル名はハッシュ ID を元に生成されたものであるため、元々のファイル情報がどこにも残っていない。

> git commit -m "add memo.md"

[master (root-commit) bb3bcb9] add memo.md
1 file changed, 92 insertions(+)
create mode 100644 memo.md

ツリー情報の ID を取得する

> git cat-file -p master

tree 74d206268a110ae68b03699a5acc3e3a23513c0c

取得したツリー情報の ID から詳細を取得する。

> git cat-file -p 74d206

100644 blob 25e878b283184ba9439cc402c26bbc84ec023da1 memo.md

25 フォルダの直下にある e878b283184ba9439cc402c26bbc84ec023da1 ファイルは memo.md と紐づいていることが確認できた。

### コミットファイルについての詳細

ツリーファイルでは、圧縮ファイルと元ファイルの紐づきは確認できるが、作成者、日時、メッセージなどの情報は取得できなかった。

> git cat-file -p HEAD

tree 74d206268a110ae68b03699a5acc3e3a23513c0c
author kyamasan <...@users.noreply.github.com> 1593529254 +0900
committer kyamasan <...@@users.noreply.github.com> 1593529254 +0900

add memo.md

ツリー情報、作成者、日時、メッセージ、(存在する場合は)親コミットの情報を確認できる。

### git init

> git init

.git ディレクトリ(ローカルレポジトリ)が作成される。
git で必要な殆どの情報(圧縮ファイル、ツリーファイル、コミットファイル、インデックスファイル、設定ファイル)はここに含まれる。

git add や git commit で作成されたファイルは全てここに含まれている。

.git 直下の objects フォルダがレポジトリの本体
.git 直下の config というファイルが git の設定ファイル

### git clone

> git clone <レポジトリ名>

git clone によって、2 つのものがローカルにコピーされる。
一つは、リモートレポジトリのファイル。
もう一つは、レポジトリ(.git ディレクトリ)

.git の情報も取得できるという点に注意。

### git add

> git add <ファイル名>/<ディレクトリ名>/.

コミットする変更を準備する為の場所がステージ。

### git commit

> git commit -m ""

git のエディタを立ち上げることなく、メッセージを追加できる。

> git commit -v

git のエディタが立ち上がって、エディタ上で変更内容が確認できる。

変更(変更、新規、削除など)を記録することを git ではコミットと呼ぶ。

### git status

コミットやステージ前にどのファイルを変更したのか確認するコマンド
以下の 2 つの情報を確認できる。

Changes not staged for commit:
① ワークツリーとステージのインデックス情報との差分
(前回ステージに追加してから変更したワークツリーのファイル)

Changes to be committed:
② ステージのインデックス情報とコミット情報との差分
(前回コミットしてからステージに追加されたファイル)

### git diff

コミットやステージ前にどのファイルのどの部分を変更したのか確認するコマンド。q 押下 で抜けることができる。

ワークツリーとステージの差分確認

> git diff

> git diff <ファイル名>

ステージとレポジトリの差分確認

> git diff --staged

### git log

変更履歴を確認する為のコマンド

> git log

一行で表示する

> git log --oneline

ファイルの変更差分を表示する(-p で特定ファイルの変更分を確認できる。)

> git log -p index.html

表示するコミット数を制限する

> git log -n コミット数

### git rm

ファイルの削除を記録する為のコマンド
レポジトリからも、ワークツリーからも、指定したファイル、ディレクトリが削除される。

> git rm <ファイル名>

> git rm -r <ディレクトリ名>

レポジトリ上のファイルは消したいが、ローカルには残したい場合は--cached オプションを使う。
(公開したくない情報(パスワードなど)を git にコミットしてしまった場合などに使う)

> git rm -cached <ファイル名>

### git mv

ファイルの移動を記録する為のコマンド

> git mv <旧ファイル> <新ファイル>

以下のコマンドと行っていることは同じ

> mv <旧ファイル> <新ファイル>
> git rm <旧ファイル>
> git add <新ファイル>

### git push

git などのリモートレポジトリにローカルレポジトリのコミットを反映させる事をプッシュと呼ぶ。

準備：リモートレポジトリの新規追加
origin というショートカットで URL のリモートレポジトリを登録する。
今後は URL を入力しなくても origin という名前で GitHub レポジトリにアップしたり取得することができる。

git clone した時にも、clone 元のリモートレポジトリも origin というショートカットに割り当てられている。
Git では、メインのリモートレポジトリには origin というショートカットを付けるのが慣例。

> git remote add origin <GitHub の URL>

> git push -u <リモート名> <ブランチ名>
> git push -u origin master

-u オプションを付けると、次回以降の push コマンドでは origin master を省略できる。

### Alias

git コマンドに別名でショートカットを付ける。
--global オプションを付けることで、PC 全体の設定を変更できる。
~/.gitconfig、~/.config/git/config にある設定ファイルに設定が反映される。

--global オプションを付けないと、project/.git/config ファイルに設定が反映される。

> git config --global alias.ci commit
> git config --global alias.st status
> git config --global alias.br branch
> git config --global alias.co checkout

### .gitignore ファイル

git でバージョン管理したくないファイル
① パスワードなどの機密情報を含むファイル
② チーム開発で必要ないファイル(キャッシュファイルなど)

- #から始まる行はコメント

- ファイル名を直接指定することで、ファイルを除外

- ルートディレクトリ(プロジェクト最上部)を指定する場合は/から書き始める。

- ディレクトリ以下を除外したい場合は、ディレクトリ名の後ろに/を付ける。

- ワイルドカードは「\*」(/以外の文字列にマッチさせたい場合)

### git checkout

ワークツリー上でのファイル変更を取り消すコマンド
git checkout はブランチの変更にも用いるコマンドで、--オプションを付けることで、ブランチを対象にしているのか、ファイルを対象にしているのか判別している。

ステージの情報を取得し、ワークツリーのファイルを上書きすることで、取り消しを可能にしている。

> git checkout -- <ファイル名>

> git checkout -- <ディレクトリ名>

全ての変更を取り消す。

> git checkout -- .

### git reset

ステージに追加した変更を取り消すコマンド
ワークツリーのファイルには影響を与えないので、ワークツリーのファイルも元に戻したい場合には、git checkout コマンドを実行する必要がある。

HEAD は現在のブランチの最新のコミットを指す。

> git reset HEAD <ファイル名>

> git reset HEAD <ディレクトリ名>

> git reset HEAD .

### git commit --amend

直前のコミットをやり直す際のコマンド
現在のステージの状態で直前のコミットをやり直す。
※リモートレポジトリに push したコミットはやり直してはダメ。(重要)

> git commit --amend

### git remote

> git remote

origin

対応する URL を表示する

> git remote -v

origin https://github.com/kyamasan/github_theory (fetch)
origin https://github.com/kyamasan/github_theory (push)

### git remote add

リモートレポジトリは複数登録できる。

> git remote add <リモート名> <リモート URL>

### git fetch

リモートから情報を取得する方法は 2 通りある。fetch、pull

git fetch を行うと、リモートレポジトリの内容がローカルレポジトリに保存される。(この時、ワークツリーの情報は変更されないので注意)
ローカルレポジトリの内容をワークツリーにも反映させる場合には、git merge コマンドを用いる。

C:\Users\..\Documents\GitHub\github_theory\.git\refs\remotes\origin

> git fetch <リモート名>

全てのブランチ情報を確認する。git fetch を行ったことで、remotes/origin/master のブランチが追加されていることを確認。

> git branch -a

- master
  remotes/origin/master

現在のブランチを remotes/origin/master に変更する。

> git checkout remotes/origin/master

remotes/origin/master の情報をワークツリーに反映させる。

> git merge origin/master

fatal: refusing to merge unrelated histories というエラーが出た場合は、--allow-unrelated-histories を付けて再実行する。
リモートレポジトリにしかないコミット情報が存在する場合(GitHub から直接ファイル追加するなど)に発生するエラー。

> git merge --allow-unrelated-histories origin/master

### git pull

リモートから情報を取得して、マージまでを一気に行うコマンド。git fetch と git merge を一度に行う。

> git pull <リモート名> <ブランチ名>

git pull origin master

=
git fetch origin master
git merge origin master

### fetch と pull の使い分け

fetch を基本的に使うべき。
git pull を行うと、自分が現在いるブランチのワークツリーにリモートレポジトリの情報を上書きする。

master ブランチにいる時に git pull origin hoge を行うと、master ブランチの内容を上書きしてしまうので、危険。

### git remote show

リモートの詳細情報を表示する。

> git remote show <リモート名>

Fetch URL: https://github.com/kyamasan/github_theory
Push URL: https://github.com/kyamasan/github_theory
HEAD branch: master
Remote branch:
master tracked
Local branch configured for 'git pull':
master merges with remote master
Local ref configured for 'git push':
master pushes to master (up to date)

### git remote rename/rm

リモート名の変更、削除を行うコマンド。

> git remote rename <旧リモート名> <新リモート名>

> git remote rm <リモート名>

### ブランチ

ブランチはコミット ID を記録したポインタに過ぎない。並行して複数機能を開発する為の機能。

HEAD やブランチの情報は.git に存在する。
.git/HEAD
.git/refs

ブランチを分岐させることで、他の人の修正に影響を受けずに開発を進めることができる。

- 復習
  Git ではレポジトリに圧縮ファイル、ツリー、コミットの 3 つの情報を持つ。コミット情報はスナップショットで、時系列順に連なっている。

ブランチとは、コミットを指すポインタの事。コミット 3 を master ブランチと feature ブランチが指している状態とする。
HEAD とは今自分が作業しているブランチを指す。

> コミット 1(ad313)← コミット 2(4t334)← コミット 3(gtb34)← master(HEAD)/feature

この状態で新たにコミットを行うと master ブランチはコミット 4、feature ブランチはコミット 3 を指す状態になる。

> コミット 3(gtb34)← コミット 4(34fry)← master(HEAD)
> ↑ feature

ここから、feature ブランチでコミット 4'、コミット 5...と別の開発を進めていくことも可能。(これがブランチの主要な役割。)

### git branch

ブランチを新規作成する為の機能。(ブランチ作成のみで、ブランチの切り替えまでは行わない)

ブランチを新規作成する。

> git branch <ブランチ名>

ローカルレポジトリのブランチを表示する。

> git branch

リモートレポジトリも含む全てのブランチを表示する。

> git branch -a

それぞれのブランチがどのコミットをポイントしているかを確認する。

> git log --oneline --decorate

HEAD が master を指し、master が 317c4b をポイントしている。GitHub の origin/master は ae99088 をポイントしていることが分かる。

317c84b (HEAD -> master) branch
e83840a branch
ae99088 (origin/master) fetch/pull
76a19b1 Create README.md
...

### git checkout

ブランチを切り替える為のコマンド

> git checkout <既存ブランチ名>

ブランチを新規作成して切り替えまで一度に行うコマンド

> git checkout -b <新ブランチ名>
