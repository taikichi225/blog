+++
author = "taiki.kono"
title = "テストページ"
date = "2021-03-01"
description = "テストページです。"
tags = [
    "flutter",
    "dart",
    "codemagic"
]
thumbnail = "images/internet_heiwa.png"

+++

テストページです。昔の記事を掘り起こして書いています。
<!--more-->
---

# はじめに
本記事は、[Flutter Meetup Tokyo #9](https://flutter-jp.connpass.com/event/126419/)でLTで話した内容に多少の追記を加えたものとなります。
LTスライドは[こちら](https://speakerdeck.com/taikichi225/flutter-for-webdegou-zhu-sitaapuriwocodemagic-plus-github-pagesdepei-xin-suru)にあります。

# Flutter for Webの発表
先日のGoogle I/O 2019でFlutter for Webのテクニカルプレビュー版が発表されました。Flutterは、Android/iOSだけでなく、Webアプリとしてブラウザで動かせるようになります。
ということで、Flutter for Webがどのようなものか早速触ってみました。

# 構成
今回は、以下の構成で検証しました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/0add3df9-6176-60d9-0865-c2d1cca7c2c3.png)

| 名前       |  概要 |
|-----------------|------------------|
| Flutter for Web | Flutter for Webは、FlutterをWebアプリとして生成する機能です。DartのコードをJavaScriptにコンパイルします。 |
| Codemagic | Flutterに特化した無料CI/CDツールです。リポジトリからソースコード取得・テスト実行・ビルド・GitHub Pagesへのデプロイを担当します。|
| GitHub Pages | GitHubの提供する静的コンテンツのホスティングサービスです。GitHubのアカウントがあれば、無料で静的サイトが公開できます。Flutter for Webで構築したアプリを配信します。 |

# 構築手順
1. Flutter for Webの導入
2. Github Pagesの準備
3. Codemagicの設定

# Flutter for Webの導入
## 1. Flutter for Webのソースコードのダウンロード
```shell
git clone https://github.com/flutter/flutter_web.git
```

## 2.ビルドツールのインストール
```shell
flutter pub global activate webdev
```

## 3.サンプルによる動作確認
```shell
cd examples/hello_world/
flutter packages upgrade
flutter pub global activate webdev serve
```

## 4.既存Flutterプロジェクトのマイグレーション
Flutterのゴールは、単一コードベースでモバイルとWebに対応したアプリケーションの構築です。しかし、今回発表されたテクニカルプレビュー版は、Flutter for Webに対応するために既存のFlutterコードを修正する必要があります。
[ドキュメント](https://github.com/flutter/flutter_web/blob/master/docs/migration_guide.md)を参考に修正します。大まかな手順は以下の3つです。

1. pubspec.yamlの修正
2. ソースコード内のimport文修正
3. webディレクトリの追加


## 5.GitHubにソースコード管理用リポジトリの作成
「flutter_web_demo」と名付けました。
この後作成するGitHub Pages用リポジトリとは別のものです。

## 6.ソースコードのプッシュ

# Github Pagesの準備
## 1.アプリ公開用リポジトリの作成
リポジトリ名に「ユーザ名.github.io」、リポジトリの種類に「Public」設定し、リポジトリを作成します。
私の場合は、「taikichi225.github.io」というリポジトリ名です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/49923d47-6dab-ac82-4948-acf550b1bc99.png)

## 2.GitHub Pages用リポジトリのクローン
```shell
git clone https://github.com/taikichi225/taikichi225.github.io.git
cd taikichi225.github.io
echo "Hello World" > index.html
git add .
git commit -m "first commit"
git push origin master
```

## 3.動作検証
以下にアクセスする。
https://taikichi225.github.io/index.html

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/1f2f8f6a-bc8d-3ebb-522f-1abe77e83f94.png)

# Codemagicの設定
## 1.Codemagicのサインアップ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/1df11480-137a-6ba6-1014-7517dfea72a4.png)

## 2.GitHubアカウントでログイン
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/0eb47b5c-6938-e2e0-9f90-4b3178a981ef.png)

## 3.ソースコード管理用リポジトリの確認
先ほどプッシュしたアプリのプロジェクトが確認できます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/8af8463a-2d78-2852-8f57-957b196dd75f.png)

## 4.ワークフローの設定

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/155589/5dc62134-39d3-64d0-00de-8a184a958132.png)

## 5.Post-build scriptに以下のスクリプト追加
GitへのデプロイはCodemagicが対応しておらず、スクリプトを書きました。

```shell

#!/bin/bash
set -ex

git config --global user.name $GIT_USERNAME
git config --global user.password $GIT_PASSWORD
git config --global user.email $GIT_EMAIL

git clone https://github.com/taikichi225/taikichi225.github.io.git

rm -rf taikichi225.github.io/$PROJECT_NAME
mkdir taikichi225.github.io/$PROJECT_NAME
cp -r $FCI_BUILD_DIR/build/ taikichi225.github.io/$PROJECT_NAME/

cd taikichi225.github.io

git remote set-url origin \
"https://$GIT_USERNAME:$GIT_PASSWORD@github.com/taikichi225/taikichi225.github.io.git"

git add .
git commit -m "deploy"
git push origin master
```

# 動作確認
https://taikichi225.github.io/flutter_web_demo/index.html

# 所感
全体的に～～～


## Flutter for Web
- テクニカルプレビュー版ゆえ、コードを多少修正したが、モバイル用に書いてきたコードがそのままWebで動いた。思っていた以上にさくさく描画してくれる。
- テクニカルプレビュー版でも未来を感じる。誰もが夢見たクロスプラットフォーム。
    - Flutter×DartだけでWebアプリを構築できるならば、CSS調整コストや変化の速すぎるJavaScriptのキャッチアップなど、Webアプリ構築関連の悩みを解決する一手になるのでは。
- デザイナーとの協業はどうするか？
    - Flutter for Webに限った話ではないが、デザイナーはDartでコーディングするの？それとも画面モックだけ作成で実際の構築はエンジニア？

## Codemagic
- GUIで簡単にフローを設定できる。
    - FlutterアプリをGoogle PlayやApp Storeにデプロイするには便利そう。
    - ビルド結果をSlackやメールへの通知を簡単に設定できる。
    - 一方で、プラグインが用意されていないため、通常とは異なる処理を含めたい場合、スクリプトをごりごり書くことになる。
- 想定以上にビルドに時間がかかる。

## GitHub Pages
- Gitの操作だけでコンテンツをホスティングできる。
    - 新しく覚えることが少なく、技術的にも気持ち的にもだいぶ楽。
- 今回設定していないが、独自ドメイン・HTTPS対応している。

# その他
今回のデモアプリ
https://taikichi225.github.io/flutter_web_demo/index.html
ホスティング用リポジトリ
https://github.com/taikichi225/taikichi225.github.io.git
アプリのリポジトリ
https://github.com/taikichi225/flutter_web_demo.git

![test](../../images/internet_heiwa.png)