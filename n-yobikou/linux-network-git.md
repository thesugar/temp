# linux・ネットワーク通信・Git/GitHub
Source: [準備しよう](https://www.nnn.ed.nico/courses/497/chapters/6889)
## 標準出力
### リダイレクト
- `>` とすると、リダイレクト（標準出力をファイルに保存）できる
- `>>` とすると、ファイルに追記して出力（元のファイル内容は保持され、その後ろに追加される）
- `2>` とすると、標準出力ではなく標準エラー出力を出力
- `2>&1` とすると、標準出力と標準エラー出力の両方を出力

```bash
ls -a > workspace/ls-output.txt
```

### パイプ
パイプは、記号 `|` でコマンド同士をつないで、一つ目のコマンドの標準出力を、二つ目のコマンドの標準入力とすることができる機能。

```bash
ls /bin | less
```

### grep コマンド

```bash
grep 含まれるかどうかを判定する単語 (検索したいファイル名)
```

第一引数に「含まれるかどうかを判定する単語」を、第二引数に「検索したいファイル名」を入力して使用する。標準入力がある場合には、第二引数を省くこともできる。  

試しに、/bin ディレクトリの中にあるファイルの中で、ss という文字列が含まれるファイルの一覧を表示してみよう。

```bash
ls /bin | grep ss
```

結果は以下のようになる。

```
bzless
less
lessecho
lessfile
lesskey
lesspipe
uncompress
zless
```

ある文字列が **含まれない** ファイルやディレクトリの一覧を得たい場合は以下の書き方になる。

```bash
grep -v ss [ファイル名 or 標準入力]
```

パイプとリダイレクトは合わせて書くことができ、たとえば以下のような形で書ける。

```bash
ls /bin | grep -v ss > [保存したいファイル名]
```

## vim
`vimtutor` コマンドでチュートリアルができる。

### vim の基本
1. カーソルは矢印キーもしくは hjkl キーで移動します。
    h (左)         j (下)         k (上)       l (右)

2. Vim を起動するにはプロンプトから `vim ファイル名` とタイプします。

3. Vim を終了するには `<ESC> :q!` とタイプします(変更を破棄)。
    もしくは `<ESC> :wq` とタイプします(変更を保存)。

4. カーソルの下の文字を削除するには、ノーマルモードで `x` とタイプします。

5. カーソルの位置に文字を挿入するには、ノーマルモードで `i` とタイプします。

6. `a` とタイプすることでテキストの追加（append）ができる。

7. なお、大文字の `I` で行頭に挿入、大文字の `A` で行末に追加。

### vim の他のコマンド
- `dw` : 次の単語まで削除
- `d$` : 行末まで削除
- `dd` : 行全体の削除
- `2dd` : 2 行削除
- `u` : undo
- `U` : 行全体の undo
- `Ctrl + R` : redo
- `0` : 行頭まで移動
- `p` : `dd` などで削除したものを貼り付け（PUT）。行の場合は、カーソルがある行の下の行に貼り付けられる。

## シェルスクリプト
- `hoge.sh` などのようにファイルを作成
- 最初の行にはシバン（shebang）を書く（「このシェルスクリプトを /bin/bash にある bash シェルで実行してほしいという意味）
- シェルスクリプトが書けたら、ファイルの実行権限を変更する必要がある。

```bash
chmod a+x hoge.sh
```

### echo コマンド
第一引数で与えられた文字列をそのまま標準出力に出力する。
以下の 2 行目の例のように、`[]` などがある場合は全体をダブルクオートで囲むなどする。

```bash
echo Please input message.
echo "Are you happy? [y/n]"
```

### read コマンド
第一引数で与えられた名前の変数に、標準入力で与えられた文字を代入する。

```bash
echo Please input message.
read message
```

これで、`message` という変数に、キーボードから与えられた値が代入される。
変数の名前の前に `$` をつけることで、変数の値を使うことができる。

```bash
read message
echo Your message: $message
```

`read -p "表示したい文字列" [入力を格納する変数]` とすることで、文字を表示しながら入力を変数に受け取ることができる。

```bash
#!/bin/bash
read -p "The 2nd highest mountain in Japan is Mt. Yarigatake, is that right? [y/n]" yn
```

### if 文

`if` と書いたあとに、条件式を `[]` で囲む。`[]` と `変数` と `演算子` の間は必ず半角スペースを入れなければならない。

```bash
#!/bin/bash
read -p "The 2nd highest mountain in Japan is Mt. Yarigatake, is that right? [y/n]" yn
if [ $yn = "n" ]; then
    echo Correct!
else
    echo Wrong!
fi
```

## 通信とネットワーク
### TCP
TCP は Transmission Control Protocol（伝送制御プロトコル）の略称。IP によるパケットを使った通信で、相手の通信状況を確認して接続を確率して、データの転送が終わったら切断するというプロトコル。相手との接続を確立しないでいきなりデータを送る方式に UDP (User Datagram Protocol) というものもある。

### ping コマンド
結果の見方について

```bash
PING www.google.co.jp (172.217.26.35) 56(84) bytes of data.
64 bytes from nrt12s17-in-f35.1e100.net (172.217.26.35): icmp_seq=1 ttl=53 time=4.82 ms
```

上記のように表示された場合、「64 バイトの情報が、**TTL (Time To Live)** と呼ばれる生存期間 53 のうちに、4.82 ミリ秒で届いた」という意味になる。この TTL はルーターを経由するたびに 1 減っていく数字で、0 になったときにはパケットが破棄される。TTL は IP の送付が無限ループに陥らないために設定されている。

### tcpdump コマンド
tcpdump は、パケットの内容を見るためのコマンド。`src` のあとに続くオプションで、送信元のサイトを限定する。`-X` オプションは、より詳細にパケットの中身を表示するというオプション。

```bash
sudo tcpdump src www.google.co.jp -X
```

## サーバーとクライアント
### tmux
tmux は、仮想端末ソフトと呼ばれるソフトウェア。1 つのコンソールで、複数のコンソールを操作したり、コンソールの状態を維持したままにすることができる。

* `tmux` : tmux を起動する。

この tmux の仮想端末から抜けるためには、`Ctrl + b` を押したあと `d` を押すことで元の画面に戻る（デタッチ）。ここからまた先ほどの仮想端末に戻る（アタッチ）には、以下のように入力する。

```
tmux a
```

### tmux の基本的な使い方まとめ

|  入力  |  説明  |  タイミング |
| ---- | ---- | ---- |
| `tmux` | 起動 | コンソール |
| `Ctrl + b → d` | デタッチ(離れる) | 仮想端末内 |
| `tmux a` | アタッチ(接続する) | コンソール |
| `Ctrl + b → c` | ウィンドウ作成(create) | 仮想端末内 |
| `Ctrl + b → 数字` | 数字のウィンドウ表示 | 仮想端末内 |
| `Ctrl + b → p` | 前の数字に移動(previous) | 仮想端末内 |
| `Ctrl + b → n` | 後の数字に移動(next) | 仮想端末内 |
| `Ctrl + b → x` | ウィンドウを閉じる | 仮想端末内 |
| `Ctrl + b → ?` | help を表示 | 仮想端末内 |

💡 tmux で入った仮想端末内の下の行（ステータス行）の見方

```
[0] 0:bash- 1:bash*                        "c35c2b8090db" 08:53 19-Apr-20
```

このステータス行は、左側から

- セッション名
- ウィンドウ名
- マシン名
- 日時と日付
を表示している。また、現在開いているウインドウには `*` が表示される。

### tmux を使ったサーバークライアント通信
tmux を起動した状態で `Ctrl + b → c` と押し、ウインドウを 2 つにする。  
これから、1 番をサーバー、0 番をクライアントにする。  
  
1 番で以下のコマンドを実行する。

```bash
while :; do (echo "Thank you for your access!") | nc -l -p 8000 ; done
```

実行すると、ずっとクライアントからのアクセスを待ち受けている状態になる。これは、シェルスクリプトで書いたサーバーである。  
`while :; do` と `done` の部分は while 句というループを行うためのシェルスクリプトの構文。終了メッセージを受け取るまでずっと、中に書いたコマンドを実行し続ける。  

`(echo "Thank you for your access!") | nc -l 8000` の部分は、nc コマンドに対して、"Thank you for your access!" という標準入力を渡している。

### nc コマンド
**nc** は NetCat という TCP や UDP の読み書きを行うコマンド。ネットワークに関して様々な場面で役立てることができるコマンドで、Web サーバーからの情報の取得や、簡易 Web サーバーの設置、メールの送信などさまざまな機能がある。  
ここでは、「8000 という **ポート** を使ってサーバーとして起動し、アクセスがあったら、標準出力の内容を返して終了する」という動作を nc コマンドで行っている。
`-l` はリッスンモード、`-p` はポートの指定。

### ポート
ポートとは、TCP や UDP の通信の取り決めのひとつで、0 番から 65535 番のいずれかの数値を設定して通信を行う。通信をするためには、必ず何かしらのポートを利用しなくてはいけない。プロトコルやソフトウェアによって、このポートが決まっているものも多くある。  
  
-------
無事サーバーが起動したところで、今度はクライアントを起動してみる。まず `Ctrl + b → 0` でクライアントのウインドウに移動する。次に

```bash
telnet 127.0.0.1 8000
```

と入力する。**telnet** を使って、127.0.0.1 という IP アドレスの 8000 番ポートにアクセスしてください、というコマンド。127.0.0.1 は自分自身を指し示す特別な IP アドレスである。

### telnet コマンド
telnet は、リモートのコンピュータにアクセスし、ターミナルのセッションを開始するためのコマンド。テキストベースの通信をすることができるが、簡単なものであれば nc コマンドのほうが簡単に実行することができる。  
nc コマンドと比べ、遠隔ログインに特化したコマンドだが、すべての通信を平文で送信してしまうため、セキュリティ上の理由から現在では ssh などの代替プロトコルを使う場合がほとんど。

### nc コマンドでチャットサービスを作ってみる
サーバーである 1 番のウインドウで `nc -l -p 8000` を実行し、クライアントである 0 番のウインドウで `nc 127.0.0.1 8000` を実行する。その後、キーボードで文字を打って Enter を押し、どちらかのウインドウの標準入力へ入力を行うと、もう片方のコンソールにも情報が表示される。

## HTTP 通信
### HTTP
HyperText Transfer Protocol: HTML などの HyperText を受け渡すための TCP/IP 上で実現するプロトコルの一つ。HTTP では、デフォルトでは 80 番のポートを使う決まりになっている。  
  
さきほどの nc を使ったサーバーとクライアントの通信では、単にテキストでやりとりしているだけだったが、HTTP では、メタ情報を持つヘッダなどを含め、一緒に送ることができる。

```bash
nc nnn.ed.jp 80
```

nc コマンドは TCP/IP の通信を開始し、待機状態になる。その状態で、

```
GET / HTTP/1.1
```

と入力してから、Enter を 2 回押し、改行を 2 つ入れてみる。これは HTTP の **GET メソッド** と呼ばれる情報取得のための宣言で、 `/` という場所、すなわちドキュメントのルートの情報を HTTP プロトコルのバージョン 1.1 で取得する、という要求。このような要求のことを **リクエスト** と呼ぶ。  

以下のような **レスポンス** が返ってくる。

```html
HTTP/1.1 400 Bad Request
Server: cloudflare
Date: Sun, 19 Apr 2020 09:59:39 GMT
Content-Type: text/html
Content-Length: 155
Connection: close
CF-RAY: -

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>cloudflare</center>
</body>
</html>
```

`HTTP/1.1 400 Bad Request` は、サーバが受け取ったリクエストが悪いリクエストだったよというヘッダ。400 と書いてあるのは HTTP のステータスコードで、不適切なリクエストだったという意味。  
`Server:` 〜 `CF-RAY:` の部分は **ヘッダ情報** と呼ばれ、送られた日付やサーバの情報、レスポンスとして返されるデータの大きさ、コネクションの状態、このヘッダに続く内容となるレスポンス本体のコンテンツ形式などが記載されている。  

`<html>` 〜 `</html>` の部分は、ブラウザに表示されるレスポンスの内容。このレスポンス本体のことを Content と呼ぶ。  

この HTTP のリクエストは TCP/IP を使い、通常のテキストでやりとりされているが、これを暗号化したものもある。それが HTTPS というプロトコルである。

### HTTPS
HTTPS は、HTTP を SSL や TLS という方式で暗号化して通信を行うプロトコル。認証局による証明書を利用して暗号化を行うことで、通信経路上の盗聴や第三者によるなりすましを防止するために用いられる。  
Web サイトなどで流出してはいけないパスワード情報や、個人情報をやりとりするページでは HTTPS で通信する必要がある。デフォルトのポートは 443 番が利用されている。

### DNS
DNS とは、Domain Name System の頭文字を取ったもの。ホスト名を IP アドレスに変換するシステムである。なお、このホスト名のことを **ドメイン** と呼ぶこともある。  

インターネット上の最上位のドメインの DNS は、13 箇所にある DNS サーバ群によって運用されている。なお、インターネット上の DNS 以外にも、特定のネットワーク内で使うためのローカルな DNS も存在する。  

また、localhost というホスト名を利用するときは内部的には 127.0.0.1 という、自分を指し示す IP アドレスに変換されるが、その設定情報は `/etc/hosts` というファイルに書かれている。

### HTTP サーバを立ててみる
1. ディレクトリを作り、その中にファイル（例：index.html）を作る
1. HTML ファイル等が用意できたら、tmux を起動して Ctrl + b → c でウインドウを作成し、作った 1 番ウインドウで `python3 -m http.server 8000` を実行するとサーバが起動する。
    - このコマンドは、Python の機能を使い、http のサーバをカレントディレクトリ `.` のファイルを使って、8000 番ポートで立ち上げるという意味になる。
1. これで、tmux の 0 番ウインドウをクライアントにしてアクセスしてみる。Ctrl + b → 0 でウインドウ 0 番に切り替えてから、`curl http://localhost:8000/index.html` と入力すると、作った HTML ファイルの内容が返ってくる。

Ubuntu を仮想マシンや Docker で立ち上げている場合でも、ポートフォワーディングをすることで、ローカル（Mac などのホスト OS）からアクセスすることができる。

## 通信をする bot の開発
### bot の動きを定義する
以下のようなシェルスクリプトを書く。

```sh
#!/bin/bash
dirname="/workspace/niconico-ranking-rss"
mkdir -p $dirname
filename="${dirname}/hourly-ranking-`date +'%Y%m%d%H%M'`.xml"
echo "Save to $filename"
curl -s -o $filename -H "User-Agent: CrawlBot; your@email" https://www.nicovideo.jp/ranking/genre/all?rss=2.0&lang=ja-jp
```

（説明）
- 保存先ディレクトリを示す変数として `dirname` を使っている
- さらに `mkdir` コマンドに `-p` オプションを与えることで、すでにディレクトリが存在する場合にも問題なく動作するようになっている
- 変数が `${dirname}` のように `{}` で囲まれている部分は、他の文字との区切りを示すため。
- バッククォートで囲んだ部分は、コマンドを実行した標準出力を文字列として取得して使う、という構文。ここでは、バッククォートの中に `date +'%Y%m%d%H%M'` と書かれている。date コマンドを実行して得られた結果を文字列として取り込むということで、結果としては `hourly-ranking-202004201430.xml` のようなファイル名が出来上がる。
- `curl` コマンドは、*何も表示せず実行するための `-s` オプション* と、*結果を指定したファイルへ保存する `-o` オプション* を指定している。さらに、`-H` オプションに続けて User-Agent を指定することで、HTTP のリクエストに UA に関するヘッダを追加している。

### bot を自動化する
このシェルスクリプトを自動的に実行するために **cron** というプログラムを利用する。  
cron とは、プログラムを決められたスケジュールにあわせて自動実行してくれるプロセスであり、週のいつ、月のいつ、何日何時何分に何を実行するのか設定することができる。  
cron を設定するには、まず

```bash
crontab -e
```

と入力する。場合によっては、エディタの選択画面が開くので Vim などを選択。
すると、`# Edit this file to introduce tasks to be run by cron` と第一行目に書かれたファイルの編集画面が出るので、その最終行に以下のように記述。

```
40 * * * * /home/workspace/bot/niconico-ranking.sh
```

これは、毎時 40 分に、指定のプログラム（`niconico-ranking.sh`）を実行せよ、という内容。これで保存すると、指定した時刻になれば自動的にスクリプトが実行される。

うまくいかない場合は、ターミナルで `/etc/init.d/cron status` と打って cron が起動しているかを調べる。起動していない場合（cron is not running と表示される）、`/etc/init.d/cron start` と打つことで起動させることができる。

## git・GitHub
### タグを使う
**タグ** とは、荷物などにつけるタグと同じで、特定のコミットの状態に別の名前をつけることを言う。`git tag タグ名` という形式でコマンドを実行することで、タグが作成される。

```bash
git tag 1.0
git push --tags origin master
```

とすることで、タグをつけたバージョンが、zip などでダウンロードできるページが自動的に作られる。