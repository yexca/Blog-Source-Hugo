---
slug: 247
title: 'DevOps を取り入れて：ブログの構築とデプロイを GitHub に任せた話'
# draft: true
author: yexca
date: '2025-05-16T18:14:06+09:00'
categories:
    - 開発実践
tags:
    - DevOps
    - GitHub Actions
    - CI/CD
---

{{< notice >}} この記事は ChatGPT によって翻訳されました {{< /notice >}}

## はじめに

最近、自分が次に何をやるか、何を学ぶかって考えてる時に、やたら目に入ってきたのが「DevOps」って言葉。最初ちょっとググったら、載ってる技術スタックはだいたい知ってるやつばっかで、「あーこれは全栈エンジニアみたいなもんか」って思ってた（まぁ実際ちょっと似てる気もする）

でも興味がなかったから放置してた（というより開発熱が冷めてた）

んで最近、4ヶ月くらい寝転がってて「やば、さすがに動かなきゃ」ってなって、またこの言葉を思い出した。改めて調べてみると……いやこれ、巻き込み力すごくね？フロントエンドとバックエンド分けても人間は一緒だし、今度は開発と運用も分離してるのに人間はやっぱり一緒、みたいな

ただ思い返すと、GitHub Action で自動化できるって話を見た瞬間に、Jekyll の時代を思い出した。あの頃は自動デプロイあったんだよね。でも自分は他のブログから移行してきて、サブフォルダ分類のクセが抜けず、それが対応してなかったから深掘りしなかった。

でも今こそ、その Hugo ブログで自動デプロイできるか試してみたくなったわけ。~~毎回コンテナから pull してアップロードするの地味にめんどくさいしね~~

## ワークフロー

ワークフローの作成は、Git リポジトリのルートにある `.github/workflows/` フォルダに YAML ファイルを置く形で定義する。ファイル名は自由だけど、今回はデプロイだから `deploy.yml` にした。

構成は大きく「名前」「トリガー」「ジョブ」の3つ。

### 名前

ここは適当でいい。とりあえず名前つけるだけ。

```yaml
name: Build and Deploy Hugo Blog
~~~

### トリガー

Action Workflow にはいろんなトリガーがあるけど、今回は記事追加したとき（Push）に動けばいいかなって感じ。

あと手動トリガーも入れておく。GitHub 側の設定ミスとかで再実行したいとき用。

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

### ジョブ

ジョブは複数定義できて、それぞれ並行で動く。今回は一個だけ。

まずジョブ名を決める。

```yaml
jobs:
  build-deploy:
```

次にどの OS 上で実行するかを指定。今回は Ubuntu。

```yaml
jobs:
  build-deploy:
    runs-on: ubuntu-latest
```

そして具体的なステップ。まずリポジトリのチェックアウト。

```yaml
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
```

続いて Hugo のセットアップ。

```yaml
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.140.1'
          extended: true
```

Hugo サイトのビルド。

```yaml
      - name: Build Hugo site
        run: hugo --minify
```

最後に GitHub Pages 用リポジトリに push。

```yaml
      - name: Deploy to BlogWeb repo
        uses: peaceiris/actions-gh-pages@v3
        with:
          external_repository: yexca/Blog-Web-Hugo
          publish_branch: main
          publish_dir: ./public
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
```

全部まとめるとこんな感じ：

```yaml
name: Build and Deploy Hugo Blog

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.140.1'
          extended: true

      - name: Build Hugo site
        run: hugo --minify

      - name: Deploy to blogWeb repo
        uses: peaceiris/actions-gh-pages@v3
        with:
          external_repository: yexca/Blog-Web-Hugo
          publish_branch: main 
          publish_dir: ./public
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
```

## トークン設定

他のリポジトリに push するにはトークンが必要。作り方は：

`Settings -> Developer Settings -> Personal access tokens -> Fine-grained tokens` に進んで、対象リポジトリに書き込みできるトークンを生成。

そのあとソースリポジトリ（俺の場合は `yexca/Blog-Source-Hugo`）の `Settings -> Secrets and variables -> Actions` の `Repository secrets` に追加。名前はワークフローに書いた通り `PERSONAL_TOKEN` にする。

## 独自ドメインの設定

GitHub Pages を独自ドメインで使うには `CNAME` ってファイルにドメインを書いておく必要がある。

でも GitHub Actions でデプロイするとき、既存のファイルが全部削除されて push されるから `CNAME` も消える。だからワークフローでファイルを生成する必要がある。

方法は2つ：

1. ワークフロー内で `CNAME` を作成：

```yaml
steps:
  - name: Add CNAME
    run: echo 'blog.yexca.net' > public/CNAME
```

1. Hugo の `static` フォルダに `CNAME` を入れておく（ビルド後そのまま出力される）

## テーマモジュールの問題

使ってるテーマが Git SubModule 経由だったから、そのまま push してもリンクだけで中身がない。自分の変更は push されないってこと。

### テーマをバックアップ

```bash
cp -r themes/Hugo-Theme-Stack tmp/Hugo-Theme-Stack-Backup
```

### SubModule の削除

初期化：

```bash
git submodule deinit -f themes/Hugo-Theme-Stack
```

Git から削除：

```bash
git rm -f themes/Hugo-Theme-Stack
```

関連フォルダを削除：

```bash
rm -rf .git/modules/themes/Hugo-Theme-Stack
```

`.gitmodules` ファイルも削除：

```bash
rm .gitmodules
```

### テーマを復元

```bash
cp -r tmp/Hugo-Theme-Stack-Backup themes/Hugo-Theme-Stack
```

テストして問題なければバックアップ削除：

```bash
rm -rf tmp
```

## JS エラーの修正

昔書いたブログ稼働時間コードで、古い書き方の 8 進数を使ってたせいで、`hugo -minify` したときにエラーが出た。

元のコード：

```javascript
Date.UTC(2021, 10, 06, 14, 15, 19)
```

修正後：

```javascript
Date.UTC(2021, 10, 6, 14, 15, 19)
```

## 結び

やっとローカルでの構築作業から解放された。Docker を使い始めた頃から、ローカル環境と開発環境は分けたいって思うようになって、ずっとコンテナで作業してた。

でも今回で完全にローカルどころかクラウドに全部任せるようになって、もはや “環境潔癖症” を克服したとも言えるかも。

で、改めて DevOps って考えてみると、これも結局、技術の進化が進みすぎた結果なのかもしれない。

昔は機械ごとに環境合わせて、アセンブリ書いて、それが高級言語になって、さらにコンテナで環境障壁が壊れて、どんどん簡単に効率良くなってきた。けど、そのたびに「入門ライン」も引き上がってんだよね。

確かに便利になったけど、同時に「ついていけなきゃ職失うスピード」も速くなってるってわけ。

まあでも、仕事は仕事、生活は生活。技術は早いけど、世界や業界の変化はそこまで速くない……はず。少しくらいは休む余裕もある、よね。
