# READ ME
[backstopjs - npm](https://www.npmjs.com/package/backstopjs)
Backstop.jsを使って、既存プロジェクト修正作業の`Visual Regression Test（以下VRT）`を行います。

##### タスク
- [ ] ua偽装が効いてるかチェック
- [ ] もう少し画像があると理解しやすくできる
- [ ] bsでも使えるようにする
- [ ] delay ,scrollToSelector をつかってwow.jsやcss.transtionに対応する

## VRT概要
VRT（画像回帰テスト）とは、Webページのスクリーンショットを以前のバージョンと比べて、ピクセルレベルでの差分を検出するテスト手法です。

### メリット
- CSSの修正で起きる意図していない表示崩れを防ぎます。
- 自動テストだから、低コストで確実に差分を検出します。
- 目視では気づけ無い軽微な差にも気づくことができます。
- 心理的な安心感と工数削減で認知的な負荷が減ります。

### デメリット
- 動的なコンテンツに対しては、追加の設定が必要です。
- 差分の追跡対象から外したコンテンツに対しては、目視・手動でのテストが必要です。
- 最初にちょっとしたコツと慣れが必要です。

## VRT導入
##### 技術選定
- 導入と利用が容易である。
- 他の案件で使いまわせるようにする。
- 既存ホームページの修正で利用する。

### 使うもの
- Node.js
- Casper.js
- Backstop.js

##### 選定理由
- backstop.jsonという1つのファイルを修正するだけで使いまわせる
- ターミナルでも3つコマンドを覚えるだけ
- 案件によっては「gulpを使っていない」もしくは「gulpのセッティングが異なる」のでgulpには組み込まない
- Casper.jsはBackstop.jsの依存先です。現段階ではあまり気にしなくてokです。

### 手順
1. ファイルをダウンロード・解凍
2. どこでもいいのでファイルを保存
3. エディタのProjectManagerなどで`vrt/node_modules/backstopjs`を登録

## VRT流れ
1. `backstop.json`を案件に合わせて更新。
2. 比較する元となる、修正前の各ページスクリーンショットを取得。
3. CSS修正（1作業分）
4. 修正後の各ページスクリーンショットを取り、修正前のものと比較。
5. 差分を確認して問題がなければ、比較元のスクリーンショットを更新。
6. すべての作業が終わるまで手順2.～5.を繰り返す。

### 1. `backstop.json`を案件に合わせて更新。
backstop.jsonファイル内の主要な部分のみ解説します。
実際に変更するのは`"scenarios"`の中だけです。
他のAPIは[backstopjs - npm](https://www.npmjs.com/package/backstopjs)を参照してください。

#### レスポンシブ設定
iPhone 5、iPad、PC画面の各ブレークポイントでスクリーンショットを取ります。
基本的に変更することはほとんどありません。
```
"viewports": [
  {
    "label": "phone",
    "width": 320,
    "height": 480
  },
  {
    "label": "tablet",
    "width": 768,
    "height": 1024
  },
  {
    "label": "pc",
    "width": 1920,
    "height": 1080
  }
],
```

#### "scenarios"を編集する
動的なコンテンツが毎回差分として検出されて鬱陶しいので、以下の各セレクタを追跡対象から外します。
```
"scenarios": [
  {
    "label": "HOME",
    "url": "[ここに修正対象のドメイン]/",
    "removeSelectors":[".loopSlider","iframe",".slider-wrap"]
  },
  {
    "label": "contact",
    "url": "[ここに修正対象のドメイン]/contact",
    "removeSelectors":[".loopSlider","iframe",".slider-wrap"]
  },
  {
    "label": "sitemap",
    "url": "[ここに修正対象のドメイン]/sitemap",
    "removeSelectors":[".loopSlider","iframe",".slider-wrap"]
  },
  {
    "label": "archive",
    "url": "[ここに修正対象のドメイン]/blog",
    "removeSelectors":[".loopSlider","iframe",".slider-wrap"]
  }
],
```

##### カルーセル
修正前後で、手動・目視のチェックが必要になります。
イレギュラーを除き、どの案件でもほぼ同じ動作をするので、「いつも通り動いてるな」くらいのチェックでok
##### iframe
インラインフレーム内のコンテンツは別オリジンのstyle.cssを参照していることがほとんどで、iframeタグや包含要素のスタイリングに気を付ければ、CSS修正作業においてiframeの中身をチェックする必要はないはずです。
##### delay
元々のページ読み込み速度が遅く、読み込み完了時間にバラつきがあるとtransitionしている要素が差分としてでてきてしまう。

### 2. 比較する元となる、修正前の各ページスクリーンショットを取得。
1. ターミナルで`vrt/node_modules/backstopjs`へcd
2. ターミナルで以下を入力
```
backstop reference
```
3. このキャプチャが比較の基準となります。

### 3. CSS修正（1作業分）
- ここで1作業分としているのは、テスト間に行うCSS修正が多いと、テスト結果が差分だらけになって見づらくなるからです。
- コツをつかむまで、「どれくらい作業してからテストするのベストか」各自調整してください。

##### コツ
- 修正範囲が広そうな修正は、短めのスパンでテストし、狭そうな修正は長めのスパンでok。

### 4. 修正後の各ページスクリーンショットを取り、修正前のものと比較。
1. ターミナルで以下を入力
```
backstop test
```
2. テスト結果がChromeの新ウィンドウで開きます。
3. 差分がでたスクリーンショットを見て意図した変更かどうか確認。

### 5. 差分を確認して問題がなければ、比較元のスクリーンショットを更新。
1. ターミナルで以下を入力
```
backstop approve
```
2. これで`backstop reference`で登録したスクリーンショットが更新されます。

## 備考
### ua偽装
ユーザーエージェントによって、
- ページを出しわけている
- 特定のDOMにクラスをつけている

場合はスクリーンショットを取っているヘッドレスブラウザのUAを変える必要があります。
##### onBefore.js
`"viewports"`のラベルが`iphone`のとき特定のUAにします。
```
module.exports = function (casper, scenario, vp) {
  casper.echo('Setting UA');
  casper.then(function () {
    if (vp.name === 'pc') {
      casper.userAgent('Mozilla/5.0 (Macintosh; Intel Mac OS X)');
    } else if (vp.name === 'iphone') {
      casper.userAgent('Mozilla /5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X)');
    }
  });
  casper.thenOpen(scenario.url);
};

```
### 画像ファイルについて
解決が必要な問題ではありませんが、1つのプロジェクトファイルを使いまわしているので、backstopjs/backstop_data/配下のbitmapsに画像データが延々とたまっていきます。
Backstop.jsの使用例としては
- 特定のプロジェクトのタスクランナーに組み込む
- ソースコード管理にVCSを使い、pushのときに差分を通知

のような例が多く、上記の問題は発生していません。
今のところは一旦、`PC掃除のタイミングでフォルダをクローンしなおす`という対処でお願いします。

