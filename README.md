# FacialExpressionMorph

ブラウザ上で 2 つの顔素材 `A` / `B` をモーフィングする実験アプリです。  
現在は次の 2 モードで構成されています。

- `morph_img/`
  - カメラ撮影または PNG 読み込みから 2 枚の顔画像を用意し、静止画モーフを確認するモード
- `morph_video/`
  - 動画ファイルまたはブラウザ内録画で 2 本の動画を用意し、同一時刻のフレーム同士をモーフするモード

ルートの `index.html` は、この 2 モードへのホームページです。

## デモ

- ホーム: [https://sooochan0809.github.io/FacialExpressionMorph/](https://sooochan0809.github.io/FacialExpressionMorph/)
- 静止画モード: [https://sooochan0809.github.io/FacialExpressionMorph/morph_img/](https://sooochan0809.github.io/FacialExpressionMorph/morph_img/)
- 動画モード: [https://sooochan0809.github.io/FacialExpressionMorph/morph_video/](https://sooochan0809.github.io/FacialExpressionMorph/morph_video/)

## できること

### 静止画モード `morph_img/index.html`

- `A` / `B` パネルごとにカメラ撮影
- `A` / `B` パネルごとに PNG ファイルを読み込み
- ライブ映像中の表情推定表示
- 撮影済み画像の表情ラベル表示
- スライダーで `A 0% - B 100%` の補間率を調整
- `A` / `B` / `Result` を PNG で保存

### 動画モード `morph_video/index.html`

- `A` / `B` パネルごとに動画ファイルを読み込み
- `A` / `B` パネルごとにブラウザ内録画
- 録画時のカウントダウン表示
- 同一時刻フレームのモーフ結果をプレビュー
- 時間スライダーで再生位置を移動
- ブレンドスライダーで補間率を調整
- モーフ済み結果を WebM 動画として書き出し
- 書き出した結果動画を保存

## 使い方

### ローカルで起動する

```bash
git clone <repository-url>
cd FacialExpressionMorph
python3 -m http.server 8000
```

ブラウザで `http://localhost:8000` を開きます。

`file://` で直接開くと、カメラ利用や ES Modules の読み込みに失敗することがあるため、HTTP サーバー経由で起動してください。

## 利用手順

### 静止画モード

1. `morph_img/` を開く
2. カメラ利用を許可する
3. `A` / `B` それぞれで `撮影` または `ファイルから追加` を使って素材を用意する
4. 下部スライダーで補間率を調整する
5. 必要に応じて各パネルの `ダウンロード` から PNG を保存する

### 動画モード

1. `morph_video/` を開く
2. `A` / `B` それぞれで `撮影` または `ファイルから追加` を使って素材動画を用意する
3. `時間` スライダーで比較したい時刻へ移動する
4. `A 50% / B 50%` スライダーで見た目を調整する
5. `書き出し` で結果動画を生成し、`保存` で WebM を保存する

## 必要環境

- カメラと動画再生に対応したモダンブラウザ
- ローカル HTTP サーバー
- インターネット接続

補足:

- MediaPipe Tasks Vision の `FaceLandmarker` は CDN と外部モデル URL から読み込まれます
- `face-api.js` の表情推定モデルは `models/` から読み込まれます
- 動画の書き出しと録画には `MediaRecorder` を利用します

## 技術構成

### 主なページ

- `index.html`
  - モード選択ページ
- `morph_img/index.html`
  - 静止画モーフ UI
  - カメラ入力
  - 表情推定
  - 顔ランドマーク抽出
  - 画像モーフ処理
- `morph_video/index.html`
  - 動画モーフ UI
  - 動画アップロード
  - ブラウザ録画
  - フレーム単位の顔ランドマーク抽出
  - 動画書き出し

### 利用ライブラリ

- `face-api.js`
  - 顔検出と表情推定
- MediaPipe Tasks Vision `FaceLandmarker`
  - 顔ランドマーク抽出
- `Delaunator`
  - 三角形分割
- `MediaRecorder`
  - ブラウザ内録画と動画書き出し

## モーフィング処理の流れ

1. `A` / `B` の画像または動画フレームを取得する
2. 顔ランドマークを抽出する
3. 中間ランドマークを補間する
4. Delaunay 三角形分割で各領域をワープする
5. `A` と `B` をブレンドして結果を描画する

静止画モードでは、加えて `face-api.js` による表情推定結果を UI 上に表示します。

## ディレクトリ構成

```text
.
├── index.html
├── morph_img/
│   └── index.html
├── morph_video/
│   └── index.html
├── face-api.min.js
├── models/
└── README.md
```

## 注意点

- 顔が正面から大きく外れるとランドマーク検出に失敗することがあります
- `A` と `B` の構図差が大きいと、モーフ結果が不自然になりやすいです
- 動画モードの書き出し形式はブラウザ依存です
- 外部 CDN と外部モデル配信に依存しているため、オフライン環境ではそのままでは動きません

## 今後の改善候補

- モバイルやタブレット向けレイアウトの微調整
- 書き出し品質や処理速度の最適化
- 対応入力形式の拡張
