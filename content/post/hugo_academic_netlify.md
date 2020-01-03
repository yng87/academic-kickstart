---
title: "Hugo Academic + Netlify でホームページを作成した"
date: 2020-01-03
lang: en-us
---

### Hugo とは

[Hugo](https://gohugo.io/) はGo言語を使って静的サイトを生成するフレームワークだ。
静的というのは、サーバーサイドでの処理を含まないということらしい。
ユーザーのリクエストに応じてレスポンスを返すというような処理はできないが、その分高速にページを表示できるようだ。

僕はHTMLもCSSもGoも完全に初心者であるが、適当に調べながらそこまで苦労せず自分のホームページを作ることができた。
その過程で学んだことを記録しておこうと思う。

### Hugo のテーマ
Hugo では作成済のテンプレートが多数存在する。とりあえずそれらの中から良さそうなものを選んで、改造して使ってみるのが良いだろう。
今回は [Academic](https://sourcethemes.com/academic/) というテーマを選んだ。
Academic は、研究に携わる人のホームページ作成を念頭に設計されている。
論文・トーク・ブログ記事などを記述するのに便利なテンプレートがあらかじめ入っているので、今回の目的にはちょうど良さそうだった。

Academic は[ドキュメント](https://sourcethemes.com/academic/docs/)が充実しているので、基本的な設定をするのにそんなに困ることはないと思う。
Getting Started と Create Content に従えば一通りサイトの personalize はできるはず。
日本語の解説記事が必要なら、[こちら](https://qiita.com/harumaxy/items/58e7e4273c61e7e260b3) が参考になる。
本記事では、僕の用途で特別に必要だったものに関してのみ、後で補足する。

僕のような初心者の場合は、そもそも Hugo がどのように動いているかをなんとなく把握しておく必要もあるかもしれない。
その場合には Hugo の [Quick Start](https://gohugo.io/getting-started/quick-start/) をやってみるとよい。

### Netlify とは
Hugo で作ったページを配信するには、Hugo がビルドしたHTMLなどのファイルを、適当なサーバーに置いておく必要がある。
静的サイトのホスティングができれば、利用するサービスは何でも良いが、ここでは [Netlify](https://www.netlify.com/) を使用する。
Hugo を公式にサポートしているので設定が簡単らしい。
確かに言われた通りクリックしていくだけで、特に詰まることなくページをビルドすることができた。

### Tips
以下では Academic を使う際の小技についてメモしておく。

#### Installation
コマンドラインでの操作に慣れている場合は、Academic の [Install with Git](https://sourcethemes.com/academic/docs/install/#install-with-git) に従ってインストールすると良い。

#### Academic の構成
とりあえず以下の4つを把握しておけば良い。
* `config/_default`: サイト全体に関わる設定ファイルが入っている
* `content`: この中に自分の論文情報やブログ記事などを入れる
* `data`: 自分で色やフォントをカスタマイズする際の設定ファイルが入る
* `themes`: ここに Academic なら Academic のテーマを作成するファイルが入っている。[テーマが提供する以上の細かいカスタマイズをしたい場合は、ここにあるファイルをコピーして使うらしい](https://sourcethemes.com/academic/docs/customization/)。

#### ローカルでの確認
初めのうちは色々と設定をいじって、それがページにどう反映されるかを見たい場合があると思う。
その時にいちいちホスティングサイトでデプロイし直すのは面倒だろう。
Hugo ではソースを編集しているローカルディレクトリで
```Bash
hugo server -p 1313
```
などどして、ブラウザで `http://localhost:1313/` を開いておくと、ローカルでの変更が随時反映されるので便利。

#### 論文リストの作成
`content/publication`に論文の情報が入った Markdown ファイルを書いていく。一つ一つ作るのは面倒なので、
1. 自分の論文を集めた`mypaper.bib`のような bib ファイルを用意する
2. そこから parse してAcademic 用の Markdown ファイルを生成する

という手順を取りたい。bib ファイルは何らかの手段で用意するとして、二段階目の Markdown ファイルを作るには、[academic-admin](https://github.com/sourcethemes/academic-admin) というプログラムを使うのが便利。
高エネルギー物理の分野では [INSPIRE](http://inspirehep.net/) というサイトが生成した bib ファイルをよく使う。
この bib ファイルをうまく扱うために[プログラムを少しだけ改造した](https://github.com/yng87/academic-admin/tree/master)。
次のようにすれば使える
```bash
git clone https://github.com/yng87/academic-admin.git
cd academic-admin
pip install -e
academic import --bibtex mypaper.bib
```
これで直下に Markdown ファイルができているはず。あとはそれを Academic の `content/publication/` に配置すれば良い。


#### Upcoming talk について
トーク情報は `content/talk` に、論文と同じように Markdown ファイルを書く。
トークの場合は、予定されている将来の情報を書くこともある。その場合Markdown ファイルの `date:` という項目に未来の日付が書かれることになる。
しかし Academic はこのままでは未来の情報を出力してくれない。
Upcoming talk を適切に表示するには、`date:` に加えて、`publishedDate:` という項目に、現在（もしくは過去）の日付を書く必要がある:
```markdown
---
title: Excellent talk
date: 2100-01-01
publishedDate: 2019-12-31
---
```
この publishedDate はトークが行われる日ではなく、サイトがアップロードされた日付を意味している。


### 終わりに
言語設定は初期から変えずに英語のままだが、僕の環境では日本語もうまく表示できている。
言語の切り替えをしたいというようなことがなければ、言語設定をいじる必要はないかもしれない。
