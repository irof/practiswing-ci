PRACSWING-CI
============================================================

## Circle CI (2017-03-12)

[![CircleCI](https://circleci.com/gh/irof/practiswing-ci.svg?style=shield&circle-token=a6a61e02339d067c28875d8b67c72cb9575ace09)](https://circleci.com/gh/irof/practiswing-ci)

設定は`circle.yml`に書くか、[ダッシュボード](https://circleci.com/dashboard)から設定するか、いずか。

何も設定しなくても`gradle test`を実行して成功失敗を教えてくれる程度には動く。
テストレポートの収集とかをしてくれないので、落ちたかどうかはわかるけれど、どのテストがなぜ落ちたかは不明な状態。
その他も微妙に痒いところに手が届かないデフォルトとなっているため、設定は必須。

ダッシュボードは手軽だが、設定はWEBのUIで行うため限定的。また、履歴も残らない。
設定できるのは`dependency`の`pre`,`override`,`post`と、`test`の`override`,`post`の5つのみ。
それぞれで実行するコマンドを設定できる。

コンテナの設定とかをいじりたいのなら、`circle.yml`に書くしかない。
[ドキュメント](https://circleci.com/docs/1.0/configuration/)読みながら適当にやる。

ダッシュボードで試して`circle.yml`に書く、とかが良いのかもしれない。
「This works great for prototyping changes.」とか書いてるし。
お試しが終わったあとはちゃんと消すこと。

### ビルド結果の収集

[ドキュメントに記載されている](https://circleci.com/docs/1.0/test-metadata/#gradle-junit-results)のをコピペ。

・・・するだけだと、テスト結果の収集に失敗する。（The following errors were encountered parsing test results:）
CircleCIが持っているgradleの出力するXMLを自分でパースできてない模様。
Gradle Wrapperを追加したらいけた（出力されるXMLが若干変わる）。

`test.override` はテストの実行が以下のように、`gradlew`があれば使用されるようになっているので不要。
```
if [ -e ./gradlew ]; then ./gradlew test;else gradle test;fi
```

これで *Test Summary* タブにテストの件数と落ちたテストの`testsuite.testcase.failure`が出力されるようになる。
あくまでサマリ表示用。

### 成果物の収集

`general.artifacts`に`/home/ubuntu/{base_dir}`以下の相対パスを記述すると、ディレクトリそのまま持っていってくれる。
GradleのテストレポートはCSSとかもまとめて欲しいので。

### 依存の解決

`dependencies.override`で依存解決のコマンドを上書き。
デフォルトで`gradle dependencies`が実行されるが、これは依存ツリーを表示するためのもので、`*.pom`しかダウンロードしてこない。
次の`database`セクションでキャッシュの保存が行われ、`/home/ubuntu/.gradle`を維持してくれるのだけれど、これではあまりに効果が薄い。
また、このリポジトリのようにマルチプロジェクト構成だと何も起こらない。

全てのキャッシュしようと思ったら`build`までしないと`runtime`やレポートでしか使わないものは入らないのだが、それをここでやっちゃうのはちょっと微妙。
なので、現実的な落としどころとして`testClasses`あたりにしておく。

なお、キャッシュはブランチ単位の模様。

### badge

`Notifications/Status Badges`でコードが生成できる。
必要に応じて変更するとよい。

`Build Settings/Advanced Settings/Free and Open Source`がONだったらトークンなくても表示できる。
OFFだったら`circle-token`をパラメーターに指定してあげる必要がある。

Free and Open Sourceはコンテナ3つ追加で使える（合計4つ）。
OFFにしたらひとつになる。

