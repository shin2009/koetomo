# KoeOkosi（声おこし）プロジェクト引き継ぎドキュメント

## プロジェクト概要

**KoeOkosi**は、通話中の音声をリアルタイムで文字に変換するPWAアプリ。
元々は聴覚が不自由な義父のために作り始めたが、一般公開・ビジネス化も視野に入れている。

- **公開URL**: https://koeokosi.vercel.app
- **GitHubリポジトリ**: https://github.com/shin2009/koetomo.git
- **ホスティング**: Vercel（GitHub連携で自動デプロイ）
- **旧URL**: koetomo.vercel.app → koeokosi.vercel.app にリダイレクト設定済み

## 技術スタック

- **フロントエンド**: 静的HTML/CSS/JS（フレームワークなし）
- **音声認識**: Web Speech API（ブラウザネイティブ）
- **翻訳**: Google Translate 無料API（translate.googleapis.com）
- **PWA**: manifest.json + Service Worker
- **データ保存**: localStorage（会話ログ履歴）
- **アクセス解析**: Google Analytics 4（測定ID: G-62SRPEPWGN）
- **Firebase**: プロジェクト作成済み（koeokosi）、現時点ではGA4のみ使用
- **デプロイ**: Vercel（GitHubのmainブランチにpushで自動デプロイ）

## ファイル構成

```
koetomo/                  ※リポジトリ名はkoetomoのまま
├── index.html            # ランディングページ（日本語）
├── en.html               # ランディングページ（英語）
├── app.html              # メインアプリ（HTML/CSS/JS全部入り）
├── privacy.html          # プライバシーポリシー（日本語）
├── privacy-en.html       # プライバシーポリシー（英語）
├── manifest.json         # PWAマニフェスト（start_url: /app.html）
├── sw.js                 # Service Worker（キャッシュ: koetomo-v8）
├── icon-192.png          # PWAアイコン（192x192）
└── icon-512.png          # PWAアイコン（512x512）
```

## 実装済み機能

### 1. リアルタイム音声認識
- Web Speech API使用
- 日本語（ja-JP）/ 英語（en-US）切り替え対応
- continuous: true, interimResults: true で連続認識
- 認識エラー時・終了時の自動再起動

### 2. リアルタイム翻訳
- ヘッダー右上の「訳」ボタンでON/OFF
- 日本語モード → 英語翻訳表示
- 英語モード → 日本語翻訳表示
- Google Translate 無料API使用（translate.googleapis.com/translate_a/single）
- 原文＋翻訳を両方表示

### 3. UI/UX
- 高齢者向けの大きく見やすいUI
- 文字サイズ4段階調整（小/中/大/特大）
- ワンタップで開始/停止
- 認識中テキスト（interim）と確定テキスト（final）の視覚的区別
- 日英UIの完全i18n対応

### 4. スピーカーフォンガイド
- 図解付きの使い方説明
- iPhoneの自動スピーカー設定案内（アクセシビリティ→タッチ→通話オーディオルーティング）

### 5. 会話ログ
- localStorageに自動保存（停止時）
- 履歴一覧表示（最新10件）
- 詳細モーダル表示
- コピー・削除機能
- 最大50件保存

### 6. 音楽パターンフィルター
- 信頼度スコア50%未満をフィルタリング
- 繰り返しパターン（ラララ、ああああ等）除外
- 1文字の無意味テキスト除外
- isLikelyNoise() 関数で実装

### 7. PWA対応
- manifest.json でインストール可能
- Service Worker でキャッシュ
- iOS Safari のホーム画面追加ガイド表示

### 8. ランディングページ
- 日本語（index.html）・英語（en.html）の2ページ構成
- ヒーロー、ターゲットユーザー、機能紹介、使い方、CTAセクション
- 右上のEN/JAリンクで言語切り替え
- フッターにプライバシーポリシーへのリンク

### 9. Google Analytics 4（GA4）
- 測定ID: G-62SRPEPWGN（Firebaseプロジェクト「koeokosi」と連携）
- ランディングページ（index.html, en.html）とアプリ（app.html）の両方に設置
- カスタムイベント:
  - `start_listening` — 音声認識開始（言語情報付き）
  - `stop_listening` — 音声認識停止（利用秒数、行数付き）
  - `toggle_language` — 日英切り替え
  - `toggle_translate` — 翻訳ON/OFF
  - `change_font_size` — 文字サイズ変更
  - `pwa_install` — PWAインストール（承諾/拒否）
  - `view_history` — 会話ログ閲覧

### 10. プライバシーポリシー
- 日本語（privacy.html）・英語（privacy-en.html）
- 現時点の内容: 音声データ非収集、localStorage保存、GA4の収集情報、Google翻訳API送信について
- Firebase Auth・クラウド保存導入時に更新が必要

## 既知の課題・未実装

### ノイズゲート（一旦無効化中）
- Web Audio APIによる音量検知のコードは残っているが、音声認識との競合問題で無効化
- initNoiseGate()、monitorVolume()、stopNoiseGate() 等の関数は残存
- 音量メーターのUIも残っている（CSSクラス: noise-gate-bar）が、startListening()から呼ばれていない
- 再実装する場合は、音声認識とマイクの同時アクセスの競合を解決する必要あり

### スピーカーフォン前提の制約
- 通話音声に直接アクセスできないため、スピーカーフォン＋マイクで外部音を拾う方式
- 通常の通話（耳に当てる）では使えない
- ユーザー自身の声も拾ってしまう

## 次のステップ（優先度順）

### 短期（すぐやれること）
1. **ethnote5-appに音声入力機能を追加** ← 次のタスク
   - KoeOkosiのWeb Speech API実装を移植
   - 写真撮影＋音声メモで簡単フィールドノート入力
   - 自動文字起こし→AIタグ付け連携

2. **KoeOkosiの会話ログをethnote5-appにエクスポート/インポートする連携機能**

### 中期
3. **音声認識精度の向上**
   - Web Speech API → Whisper API / Google Cloud Speech-to-Text への移行
   - ストリーミング認識対応

4. **話者分離（Speaker Diarization）**
   - 自分の声と相手の声を区別
   - Google Cloud Speech-to-Text のspeaker diarization機能

5. **ノイズゲートの再実装**
   - 別アプローチ：Web Audio APIのMediaStreamを音声認識と共有
   - または音声認識結果の後処理として実装

### 長期（ビジネス化）
6. **ユーザー認証・クラウド保存**（Firebase予定）
7. **有料プラン（フリーミアム）**
   - 無料：基本文字起こし
   - 有料：翻訳、ログ無制限、高精度認識
   - 聴覚障害者は完全無料
8. **多言語対応**（中国語、韓国語等）
9. **法人向けプラン**（病院、コールセンター等）
10. **補助金申請**（厚労省、NEDO、総務省等）

## ビジネスコンテキスト

### 開発者
- SHIN（odagiri）
- 会社: クラックス（crux-lab.co.jp）- 食品EC運営
- 関連プロダクト: ethnote5-app（AIエスノグラフィプラットフォーム）
- EMA（Ethnography Marketing Association）を設立予定

### ターゲットユーザー
1. 聴覚障害者・難聴の高齢者（無料提供）
2. 英語学習者（有料）
3. 海外取引のあるビジネスパーソン（有料）
4. 医療・介護施設（B2B）
5. 観光・接客業（B2B）

### EMA連携計画
- KoeOkosiモニター募集をEMA経由で実施
- ethnote5-appで行動記録を収集
- KA法で価値抽出 → ケーススタディ化
- モニターからの口コミでユーザー拡大

### インバウンド観光展開構想
- 来日外国人にKoeOkosiを使ってもらい、観光体験データを収集
- KoeOkosi（翻訳ツール）× ethnote5-app（体験記録）× EMA（分析）の一気通貫
- ターゲット: 自治体観光課、ホテルチェーン、鉄道会社等
- モニター募集: ゲストハウス提携（宿泊費割引と引き換え）

## 開発環境メモ
- Windows PC（C:\Users\odagiri\Desktop\koetomo）
- VS Code使用
- Git / GitHub連携済み
- Vercel Hobbyプラン（無料）
- Firebaseプロジェクト: koeokosi（Sparkプラン・無料）
