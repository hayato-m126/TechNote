# はじめに
Asciidocは、Markdownとほとんど同じような記述法でお手軽にかけて、規模の大きい文章を書くことにも適しています。

例えば、[Pro Git](https://git-scm.com/book/ja/v2)は、日本語版PDFで500ページを超えるGitの解説書ですがAsciidocで書かれています。

ソフトウェアの設計書など、そこそこの規模になる技術文章を記述するのにAsciidocが良いと思い導入してみました。
参考にしたページの紹介と、実際に使ってみて、感じた利点などを紹介します。

# 参考
以下のリンクのComparison by exampleにMarkdownの記法と比較して、どう書くかが記載されているので参考になります。

[AsciiDoc vs Markdown, asciidoctor docs](https://asciidoctor.org/docs/asciidoc-vs-markdown/)

少し分量が多いですが、日本語化されたリファレンスもあります。

[Asciidoctor 文法クイックリファレンス(日本語訳)](https://takumon.github.io/asciidoc-syntax-quick-reference-japanese-translation/)

とりあえず、Markdownの書き方の違いを確認すれば書き始められるでしょう。

# 利点
- テーブルの結合が使える。テーブル内で箇条書きや、画像挿入等も可能
    - USDMなどを書くのに便利
- 動画を直接埋め込める
    - Markdownではアニメーションgifを活用するのが一般的。asciidocならyoutubeを直接埋め込める
- 外部ファイルをincludeできる。
    - asciidocのファイルを分割して他のファイルから読み込める。TeXのように使える。長いドキュメントを作るにはほぼ必須機能。
        - Pro Gitはチャプターごとに分割して作られていた。
    - ソースコードを読み込んで文章にそのまま貼れる
    - dotファイル、PlantUMLなどの図の元となるテキスト形式ファイルを読み込める
- ブラウザの拡張機能で簡単にプレビュー環境をつくれる。([別記事参照](https://qiita.com/hyt126/items/9503ac50ab72553d1032))
    - Githubなどのバージョン管理サーバー上での文章レビューがやりやすい。レビューワーにも優しい
- vscodeでリアルタイムプレビュー付きの編集環境を作れる([別記事参照](https://qiita.com/hyt126/items/fdeff36f09bb221dfac0))
- asciidoctor-pdfでそこそこな見栄えのPDF文章を出せる
    - 仕事で文章を残すのに、表紙だけワード等の指定フォーマットで作って合成できる。
- テキストなのでバージョン管理しやすい
    - ソースコードと一緒に設計文章もgitで管理できる

上記のような利点があり、ソフトウェアの設計文章を作るのに向いていると思っています。
Markdownより高機能で、TeXよりもお手軽な文章作成環境という印象です。

# 現状認識している欠点
使っていて、以下の点が気になりました。

- 画像の読み込みが相対パス指定
    - includeで別ファイルを読み込んだ場合は、呼び出し側のファイルから相対パス

例えば、Pro Gitのフォルダ構成は以下のようになっており、ルートにあるチャプターファイルで章の中のセクションファイルをincludeしています。

```
│  チャプター1.asc (この中でbook配下のセクションを読み込んでいる)
|  チャプター2.asc (この中でbook配下のセクションを読み込んでいる)
|  ・・・
|
├─book
│  ├─チャプター1フォルダ
│  │  └─セクションフォルダ
|  |        セクション1.asc
|  |        ・・・
│  ├─チャプター2フォルダ
│  │  └─セクションフォルダ
|  |        セクション1.asc
|  |        ・・・
|
├─images
|       すべての画像ファイル
|
```

しかし、セクションのファイルから読み出す画像ファイルの指定は、チャプターからの相対パスなので、画像の読み込みはimage::images/xxx.pngのような形で指定します。
これだと、セクションファイルを単体で開いたときに自分の配下にimagesフォルダはないので読み込みが出来ません。
TeXに慣れていると、この辺の動作の違いに戸惑います。

せっかく、include機能でファイルを分割して作成できるのに、画像が読み込み側からの相対パス指定となると、分割したファイルがどうincludeされるのか意識して、パスの指定をする必要が出てきてしまいますので、この仕様はいまいちだと思っています。

imagesdirコマンドで、画像を読み込む基準のディレクトリを変更することが出来ます。都度ディレクトリを指定するということも考えられますが、asciidocのファイルと、使用する画像は同じフォルダに置くか、サブフォルダに画像フォルダを作るといったように、予めフォルダ構成を考えた方が懸命かと思います。

# 拡張子何にするか
adocが良さそうです。
理由は[ここ](https://qiita.com/hyt126/items/9503ac50ab72553d1032#%E6%8B%A1%E5%BC%B5%E5%AD%90%E6%AF%8E%E3%81%AE%E5%8B%95%E4%BD%9C%E3%81%AE%E9%81%95%E3%81%84)

# 仕事で使う場合の文章作成プロセス案
以下のような感じでやれば、文章の改定も管理出来て、良いかなと思っています。

1. バージョン管理サーバーのmasterブランチからpullする
2. 編集用にブランチを切って，文章を作成する
3. 完成した文章をコミット＆プッシュ
4. バージョン管理サーバー上でプルリクエストを投げる
5. バージョン管理サーバー上で承認行為を行う。
6. 承認者が内容を確認したら，ワードやエクセル製の社用の独自の表紙に記名日付を入れて表紙のPDFを保存する
7. 承認者が表紙をコミット＆pushし，マージする。
8. Jenkinsのジョブで自動でPDFを作成する。
9. 出来上がったPDFを適当な場所に配布(PDF合成のジョブの中で一緒にやる)

# asciidoctor-pdfで独自の表紙を合成するコマンド
本体はsample.adoc、表紙はcover.pdfの場合

```
$ asciidoctor-pdf ros_install.adoc -o ros_install.pdf -a notitle -a front-cover-image=image:cover.pdf[] -n -r asciidoctor-diagram
```

# サンプル
私が作ったサンプルを以下においています。
https://github.com/hayato-m126/AsciidoctorSample