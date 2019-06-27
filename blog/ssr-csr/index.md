# Web 上でのレンダリングハイパフォーマンスレンダリング

Topics

- the concept of CSR, SSR, and their changes
- some metrics such as FCP, TTI
- HTML のレンダリングとローディングがサイトパフォーマンスにどれだけ影響するか

# CSR と SSR の狭間で 〜ハイパフォーマンス Web サービスを夢見るフロントエンドエンジニアの闘い〜

## Abstract

2019 年の Web エンジニアが Web サービスのパフォーマンスを上げるために何を頑張っているのかを説明する。
発展的なトピックについては Rendering on the Web の内容と同じで、そこに至るまでの Web 開発の歴史を Web に詳しくない学生向けに説明する。
時間は 5 分なので実際短い。

## Outline

- パフォーマンス改善の歴史
  - Growing Web Technologies
    - 2005: jQuery → 簡単に動的なサイトが作れる
    - 2010: MVC, MVVM, MVP → 「現在の状態」が管理できる
    - 2015: Flux, vDOM → 状態の「更新」がきれいにできる
    - 2020: TypeScript, TDD → 巨大なコードベースでも開発の見通しが良くなる
    - ページをリッチにする → コードベースが巨大になる → 巨大なプログラムを見通しよく開発するための技術が発展する
      - これの繰り返し
  - ページあたりのデータ量は年々増加している(HTTP Archive)
    - ページの機能が増えたのでデータ量は必然的に増える
    - 増えた機能を管理するためのコードでコードベースは更に巨大になる
  - 多くのデータを転送するため、ネットワーク速度とは常に競争状態
  - クライアントの CPU も使うようになってきてパフォーマンスの問題も顕在化
  - まとめ: FCP や TTI を早くしつつ立派なアプリケーションを提供しなければ行けないというのが今日の Web エンジニアの課題
- SSR と CSR
  - クライアントでいろいろやるからパフォーマンスが問題になる
    - 逆にサーバーで React を動かしてしまえば良いんだと考える
      - サーバーのリソースはコントローラブル
    - HTML だけ送信すればよくなるのでリソース送信量も減少する
      - vDOM 技術は基本的に HTML を出力するための技術 → HTML が出力できれば動かすのはクライアントでもサーバーでも関係ない
  - Server Side Rendering: React などをサーバーで動かして HTML を送信する
  - Client Side Rendering: React などをクライアントで動かして JS を送信する
    - ページの枠だけ SSR で中身は CSR にするとか、最初のページは SSR でだけど遷移先は CSR にするとか、中間的な戦略がいろいろある
  - パフォーマンスの話をするのでメトリクスを説明する
    - TTFB(Time to First Byte) → FP(First Paint) → FCP(First Contentful Paint) → TTI(Time to Interactive)
  - Static SSR
    - デプロイ時に HTML/CSS をビルドしてリソースを静的配信する
    - :+1: サーバーの責務は静的ファイルの送信だけ
    - :+1: サーバーは置いてあるコードを返すだけなので FCP は最速
    - :+1: JS もないので TTI も最速
    - :-1: すべてのページをデプロイのたびに作り直さないといけないので、コンテンツが限定的でないと使えない
    - 対象 → コンテンツが決まっている静的サイト → ヘルプページやポートフォリオなど
  - CSR with Pre-rendering
    - ユーザー固有の情報を除いた App Shell の部分だけを事前にレンダリングしておいて、ユーザー固有部分は CSR を使う
    - :+1: サーバーの責務は静的ファイルの送信だけなのでツールを使えばそんなに難しくない
    - :+1: 速い FP
    - :-1: 遅い TTI
    - 対象 → アプリっぽいサイト
  - SSR with Hydration
    - サーバーとクライアントで 2 度レンダリングを行う
      - サーバーでは HTML の作成のため
      - クライアントではあらゆる JS 処理の作成とイベントハンドラーの登録のため
    - :+1: 速い FCP
    - :-1: 遅い TTI → あるのに反応しない → 不気味の谷
- 今年イチオシの技術
  - Streaming SSR
    -HTML をチャンクで送信しブラウザに到着し次第徐々にレンダリングする
    - :+1: 最速の FP
    - :+1: 速い FCP
  - Progressive Hydration
    - Hydration を徐々に行う（Streaming SSR が HTML のストリーミングなら Progressive Hydration は JS のストリーミング）
    - ビューポートに入った要素から順番に処理をダウンロードしてイベントハンドラーに登録する
    - :+1: 最初のページを表示するのに必要な JS の絶対量が減るので TTI を早めることができる → 不気味の谷の解消
    - :-1: 一般に JS の処理はコンポーネントをまたいで複雑に絡み合っているので実装が難しい
      - コンポーネントごとにきれいに処理を分けることができていれば簡単？
    - :-1: 現在は React でライブラリレベルでのサポートはない
      - IntersectionObserver と ReactDOM.hydrate を駆使することで Hydrator 層を自作するすることはできる
- まとめ
  - サービスの特性を鑑みて適切に選ぼうな

## Old

- Static Pages の時代
  - ファイルが
- Pre-rendering の時代
  - サーバーで HTML を生成してユーザーごとにカスタマイズした結果を返す
- CSR の時代
- SSR の時代
- CSR と SSR の選択の時代
- React, Vue, Angular, and other JavaScript libraries generates HTML
  - Why? → Rich, Scalable

* React や Vue といったフロントエンドフレームワークが流行してユーザーインターフェイスの責務がフロントエンド（ブラウザ）で完結するようになってきた
  - 2011 年から 2019 年でデスクトップ向けサイトのリソースサイズは 4 倍に増加した
  - モバイルは 11 倍に増加した
  - ネットワーク環境の改善（機会的理由）
  - React,Vue 等のフロントエンドフレームワークの興隆によりフロントエンドだけで複雑なアプリケーションを構築できるようになった
  - JavaScript の実行時間も増加
  - リッチなアプリケーションを作れるのは良いがネットワーク・実行時間の両方で問題が顕在化
* SSR か CSR か
  - main と bundle.js しかない典型的な CSR の React アプリケーションを紹介
    - 要はブラウザで HTML を構築しているだけ（レンダリングの定義）
  - SSR の場合はこれをサーバーサイドで行う
    - クライアントでの JavaScript の実行時間を削減できるが TTFB の低下を引き起こす
    - サーバーサイドの方が挙動をコントロールしやすいので JS の実行時間を本質的に削減できる
      - コンポーネントキャッシュを異なるユーザー間で共有できたり
  - Pros/Cons があるので選択が重要だが中間的解決法もある
  - Static SSR
    - すべてのページを事前にレンダリングしてしまう
  - CSR with Pre-rendering
    - App shell など基本的な部分だけレンダリングしてあとは JS として配信する
* 最新の解決方法として Streaming SSR と Progressive SSR を紹介する
  - SSR は TTFB が遅い
  - SSR with Hydration
    - サーバーサイドでは HTML の構築だけを行う
    - イベントハンドラの設定等のロジック的な部分をクライアントサイドで行う
    - TTI >>> FCP（不気味の谷）
  - ## Streaming SSR

## SSR

- JavaScript をクライアントへ大量送信することを防げる
  - First Paint (FP) と First Contentful Paint (FCP) と Time to Interactive (TTI) が向上
- サーバーでの処理に時間がかかる
  - TTFB が悪化
- React では renderToString が同期処理でシングルスレッドのため遅くなることがある
- コンポーネントのキャッシュ・メモリ消費の管理・メモ化の適用等が必要

## 静的レンダリング

- レンダリングがビルド時に行われる
  - 最速の FP, FCP, TTI を提供する
- Gatsby, Jekyl, Metalsmith
- 多くの固有ページを持つサイトでは使えない
  - Advanced: [AMP Cache](https://amp.dev/documentation/guides-and-tutorials/learn/amp-caches-and-cors/how_amp_pages_are_cached)
  - Advanced: signed html exchanges

## CSR

- JS のダウンロードと実行のタイムライン
- インタラクティブなページは SSR の効果が薄い
- ページが大きくなってくると処理同士が互いの処理能力を奪い合うようになってくる
  - 積極的なコード分割が必要になる
  - JS の遅延ロード

## SSR with Hydration

- FP は速くなるが TTI に重大な問題を引き起こすことがある
  - 見た目は存在するのにクリックしても何も起こらない
- データとビューが二重で描画されたりする

## Streaming SSR

- HTML をチャンクで送信しブラウザが順次表示する
