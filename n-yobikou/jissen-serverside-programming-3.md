# 実践サーバーサイドプログラミング（3）
[Source](https://www.nnn.ed.nico/courses/497/chapters/6891)

## § 15. 「予定調整くん」の設計
ここまで、Web プログラミングに関する様々な知識を身につけてきた。ここからは入門コースの集大成となる Web サービスを開発していく。

入門コースの仕上げとして、スケジュール調整サービスを作る。  
スケジュール調整サービスの基本的な要求は一つ。

- できるだけ多くの人が参加できる適切な予定日時を決定したい

以上である。  
具体的なユースケースとしては、

- GitHub アカウントを知っている仲間内で、オフ会の日程を決定する

など。**ユースケース** とは、実際にシステムがユーザーに利用される際のやりとりのことを言う。

まず最初にこのサービスの名前とプロジェクト名を決める。  
サービスの名前は「予定調整くん」で、プロジェクト名は `scheduke-arranger` とする。

### 「予定調整くん」の要件定義
予定調整くんに必要な機能は以下のとおりとする。

- 予定を作れる
- 予定に候補（候補日）が作れる
- 予定の候補に対して出欠を編集できる
- 予定に対してコメントが編集できる
- 予定を削除できる
- 予定を編集できる

以上のような要件は **機能要件** とも呼ばれ、要求を満たすための機能があることを定義したものとなっている。

機能要件でないものを **非機能要件** という。非機能要件には、機能に付随する性能に対する要件や、セキュリティに関する要件などがある。

### 「予定調整くん」で出てくる用語を定義する
それでは、要件にあがった用語を細かく定義していく。

|用語|英語表記|意味|
|--|--|--|
|ユーザー|user|予定調整くんの利用者|
|予定|schedule|複数のユーザーの出欠が必要なイベント|
|候補|candidate|予定の開催の日時および期間の候補、つまり候補日程|
|出欠|availability|予定の候補に対する出欠意思で、「欠席・わからない・出席」が選択できる|
|コメント|comment|予定に対してユーザーがつけるコメント|

以上のように、この「予定調整くん」野中に登場する言葉とその意味を定義する。

このように、システムの中に登場する用語と意味をしっかり定義するということは、ソフトウェアづくりの中で非常に重要！

### データモデリング
今度はそれぞれのデータモデリングを行い、データモデルを作成しよう。

先ほど紹介した用語はすべて、データベース上におけるエンティティとみなしてよいだろう。

エンティティ同士の関係を表すと以下の ER 図のようになる。

<img src="./images/schedule-arranger-er.png">

主キーや外部キーなどの属性はすべて省いて、エンティティ名だけで書いている。  
一番中心となるエンティティはユーザー。  
ユーザーは、予定、出欠、コメントとそれぞれ「1 対 多」の関連を持っている。

- ユーザーは、予定を作成できる
- ユーザーは、出欠を編集できる
- ユーザーは、コメントを編集できる

以上の要件があるため、この関係は妥当だと言える。  
またこの関係から、予定、出欠、コメントはそれぞれ、ユーザーに従属している。

そのためこれらの関係を表す線には、 `1` と 0 から N を表す `0..n` が記載されている。この関係は「1 対 多」の関連における従属を表す ER 図の線の書き方である。

次に予定は、候補とコメントとそれぞれ「1 対 多」の関連を持っている。これも、

- 予定には、候補がある
- 予定には、ユーザーによってコメントが編集できる

これらの要件を満たすために必要な要件であると言える。同様に、候補とコメントは予定に従属している。

そして候補は、出欠と「1 対 多」の関連を持っている。

- 候補に対して、ユーザーは出欠を編集できる

という要件があるためこれも必要だろう。  
同様に出欠は候補が存在しないと記入できないため、これも出欠が候補の従属エンティティだということができるだろう。

さて、このようにデータモデルができたところで、先ほどの基本的な要件（「予定を作れる」など）が満たされるかを確認すると、どうやらこのデータモデルでよさそうだとわかる。

なお、このようにデータモデルなどの設計を行っているが、実際に実装で都合が悪いところが出てきたら、それにあわせてデータモデルはへんこう　していく。

このような設計は、設計どおりにしっかりと作ることが重要なのではなく、設計をすることによって、要件に漏れがないかといったことや、根本的な仕組みに問題がないかをチェックすることが目的である。

したがって、設計をしたら感あらずこの通りに作らなくてはいけない、というわけではない。

### URL 設計
次に要件を満たすための URL 設計を行おう。  
まずは内容を表示するページ構成を考える。

- トップページ
- 自分が作った予定の一覧表示ページ
- 予定表示ページ
- 予定作成ページ
- 予定編集ページ
- 出欠表ページ
- コメントページ

思いつくままに挙げてみた。ログインとログアウトのページは省いている。

ただし、機能的に一緒であっても問題なさそうなページがありそうなので、まとめてみる。

- トップ/自分が作った予定の一覧表示ページ
- 予定表示/出欠表/コメントページ
- 予定作成ページ
- 予定編集ページ

このようにまとめられそうである。  
トップページでは、自分が作った予定の一覧をログイン時とログアウト時で出し分けるように実装すればよさそう。

予定の内容、出欠表、コメントは同じページ内にまとめて出欠やコメントの編集を Ajax を利用して編集するとより利便性が上がる。

予定の作成と編集ページは、普通の HTML のフォームdえ実現できそう。

これらを踏まえて、メソッドと URL のパス、内容をまとめると以下の表のようになる。

#### ページの URL 一覧

|パス|メソッド|ページ内容|
|--|--|--|
|/|GET|トップ/自分が作った予定の一覧表示ページ|
|/schedules/new|GET|予定作成ページ|
|/schedules/:scheduleId|GET|予定表示/出欠表/コメントページ|
|/schedules/:scheduleId/edit|GET|予定編集ページ|
|/login|GET|ログイン|
|/logout|GET|ログアウト|

以上のようなページの URL で表現できそう。  
表のパスに出てきている `:scheduleId` は、予定エンティティの主キーとする。

続けて、ページではなく、フォームの投稿先や Ajax で利用する Web API の URL も設計してしまおう。

#### Web API の URL 一覧

|処理内容|パス|メソッド|利用方法|
|--|--|--|--|--|
|予定作成|/schedules|POST|フォーム|
|予定編集|/schedules/:scheduleId?edit=1|POST|フォーム|
|予定削除|/schedules/:scheduleId?delete=1|POST|フォーム|
|出欠編集|/schedules/:scheduleId/users/:userId/candidates/:candidateId|POST|Ajax|
|コメント編集|/schedules/:scheduleId/users/:userId/comments|POST|Ajax|

以上のようにまとめられる。メソッドはすべて POST とした。

なおここに登場する `:userId` はユーザーエンティティの主キー、`:candidateId` は候補エンティティの主キーとする。

このようにデータモデルと URL 設計をすることで、Web サービスの全貌が見えてくる。

ただし、特に URL 設計に関しては、適切な URL 設計が実装前にはできないことも多い。セキュリティの要件やパフォーマンス上の要件などの技術的制約により URL 設計が影響をうけることがあり、それらは詳細な実装の前には想定しづらいためである。

そのため、場当たり的にどんどん作っていきたいという考えもあるが、とはいえ何も考えなしに作成していくと全体の URL 設計の統一がなされず非常にわかりづらい URL 構成になってしまいがち。

以上のように大枠の URL 設計を経ることによって、全体として統制が取れ、わかりやすい URL を設計することができる。

なお Ajax などで利用する Web API の設計には様々な設計思想やツールがあるが、ここでは紹介を割愛する。

### モジュール設計
ここまではインターフェースの設計であった。  
ここからはモジュールの設計も行っていく。

ただし Web フレームワークの Express を利用する前提に立つことでずいぶんとモジュール設計コストを下げることができる。

URL 設計にそって、`/routes` ディレクトリ以下の Router モジュールを以下のように用意すればよい。

### Router モジュール一覧
|ファイル名|責務|
|--|--|
|routes/login.js|ログイン処理|
|routes/logout.js|ログアウト処理|
|routes/schedules.js|予定に関連する処理|
|routes/availabilities.js|出欠の更新に関する処理|
|routes/comments.js|コメントの更新に関する処理|

このように責務を分割しよう。

それぞれの責務は独立性と凝集性が高く、分離しやすいため、上記のようなモジュール構成で作ることにより、効率的に開発することができる。

このように Web フレームワークを用いることで、お決まりのモジュール構成の形式にしたがって設計を考えられるため、設計の段階でもずいぶん楽をすることができる。

なお、データモデルやデータストアへの永続化に関しては、`/models` というディレクトリを作成し、それぞれのエンティティごとにファイルを定義する。

### データモデル一覧

|ファイル名|責務|
|--|--|
|models/user.js|ユーザーの定義と永続化|
|models/schedule.js|予定の定義と永続化|
|models/candidate.js|候補の定義と永続化|
|models/availability.js|出欠の定義と永続化|
|models/comment.js|コメントの定義と永続化|

今回は永続化には sequelize を用いる。  
上記の表では、データモデルで決定したエンティティの設計に沿ってモジュール分割を行っている。

これで JavaScript で利用するモジュールは `/routes` と `/models` ディレクトリ以下にそれぞれモジュール分割することができた。

なお、このように、リクエストのルーティングを行い処理をコントロールする部分と、エンティティのモデリングや振る舞い、永続化などの処理をする部分と、表示内容の形式を定義する部分を分割する構造のことを MVC という。

### MVC
<img src="./images/mvc.png">

M は Model、すなわちデータモデルであり、ここでいう `/models` ディレクトリ以下のモジュールである。  
V は View、ここでは `/views` ディレクトリ以下のビューという見た目を司る pug テンプレートとなる。  
C は Controller、Express では `/routes` ディレクトリ以下の Router モジュールを表す。

なお世の中には、以上のような MVC の構成をデフォルトで提供している Web フレームワークも存在する。

ただし、MVC は、いくつか派生したアイデアがさまざまな書籍で紹介されていることもあり、 M, V, C それぞれの責務や依存関係のあり方については多くの解釈がある。

そのため共通項としては、M, V, C のそれぞれのモジュールに分割された構造を持つ設計だという解釈でよかろう。

なお、このように、モデル、コントローラー、ビューをそれぞれモジュールとして分割して、 インタフェースを制限することで、 それぞれのモジュールの交換性能を高め、変更時の影響を少なくできるというメリットがある。

半面、モジュール分割しない場合よりも、モジュール間のインタフェースの定義の実装量は増えてしまいがち。

なお今回の構成では、ビューはモデルに依存していますが、モデルはビューに依存しないため、 テンプレートエンジンを仮に変えたとしても、`/models` 以下のモジュールに変更を加えることなく利用し続けることができるというメリットがある。

以上で、予定調整くんの設計は完了となる。

💡 ここで、GitHub 認証を使うためのアプリケーションを作成しておこう。

GitHub へのログイン後 `https://github.com/settings/applications/new` にアクセスし、

- Application name を、予定調整くん（開発用）
- Homepage URL を、`http://localhost:8000/`
- Application description を、GitHub 認証を利用して予定を調整してくれるアプリケーション
- Authorization callback URL を、 `http://localhost:8000/auth/github/callback`

以上に設定してアプリケーション登録をする。