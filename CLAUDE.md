# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイドです。

## プロジェクト概要

忖度リバーシ — ブラウザで遊べるリバーシゲーム。選択したモードに応じてCPUがプレイスタイルを調整する。素のHTML/CSS/JavaScript（フレームワーク・ビルドツール・パッケージマネージャなし）で構築。Firebase HostingとFirebase Realtime Database（週間ランキング用）でホスティング。

## コマンド

```bash
firebase serve          # ローカル開発サーバー起動
firebase deploy         # 本番環境へデプロイ（Firebase Hosting + データベースルール）
```

ビルド・テスト・リンターは未設定。アプリは `public/index.html` 単一ファイルにCSS・JSが埋め込まれている。

## アーキテクチャ

### ファイル構成

- `public/index.html` — アプリ全体（約1860行）: HTML構造、CSSスタイル、ゲームロジックすべてを含む
- `public/sw.js` — Service Worker（アセットはキャッシュ優先、ナビゲーションはネットワーク優先）
- `public/manifest.webmanifest` — PWAマニフェスト
- `firebase.json` — ホスティング設定（全ルートをindex.htmlにリライト）
- `database.rules.json` — 週間ランキング用のRealtime Databaseセキュリティルール

### ゲームモード

- **忖度モード (sontaku)**: CPUが意図的に手加減し、プレイヤーがちょうどよく勝てるように調整する
- **通常モード (normal)**: CPUがminimax探索で全力プレイ

`sontakuMode` ブール値フラグでCPUの着手選択ロジックが切り替わる。

### AIエンジン（index.html内）

**盤面表現**: 64ビットBigIntによるビットボード。2つのビットボード（`P`: 手番側、`O`: 相手側）で盤面全体を表現。ビット0 = A1（左上）、ビット63 = H8（右下）。

**主要関数**:
- `legalMoves(P, O)` — 8方向のシフト演算で合法手を生成
- `flipsForMove(P, O, move)` / `applyMove(P, O, move)` — 着手実行
- `evalBoard(P, O)` — 多要素の局面評価（石数、手数（モビリティ）、隅、X打ち、確定石、フロンティア、偶数理論）
- `phaseWeights(occupied)` — 序盤/中盤/終盤の評価重みをブレンド
- `minimax()` — 再帰探索（深さ3）、置換表付き（`searchTT`、上限25万エントリ）
- `solveExact()` — 空きマス10以下（`ENDGAME_SOLVE_EMPTY`）で完全読み切り
- `chooseSontakuMove()` — 忖度モードの着手選択（全手を評価し、接戦になる手を選ぶ）

### ランキングシステム

Firebase Realtime Databaseに週間ランキングを保存（キーは日曜始まりの週開始日）。データベースルールによりモードが `"sontaku"`、勝利が `true`、名前1〜20文字であることを検証。スコア昇順（低いほど良い）、タイムスタンプで同点解決。上位5件を表示。

### コーチングシステム

各対局後、`worstPlayerMistake`（最大の悪手）と`bestPlayerGoodMove`（最大の好手）を追跡（最善手との評価値差をゲームフェーズで重み付け）。推奨手を盤面で可視化して表示。

## 言語

UIテキスト・コメントはすべて日本語。
