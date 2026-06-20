# 02 reverse-engineer-song — 既存曲の定性RE

> YouTube動画（楽曲）をリバースエンジニアリングし、音楽/画像/動画の3モダリティの生成AI用プロンプト仕様書を出力。

---

## できること

- YouTube楽曲を分析し、**生成AIで再現するためのプロンプト仕様書**を出力
- 3モダリティ（音楽 / 画像 / 動画）を言語で記述
- 定性分析（「どんな曲か」を言葉で）

## トリガー

「逆コンパイルして」「この曲から仕様書作って」「リバースエンジニアリング」「YouTubeから分析して」「/reverse-engineer-song」

## 設計の核心: Gemini に委任

Claude Code は **YouTube を視聴できない**。そのため、Gemini API（`scripts/api/gemini.py`）経由で分析を委任し、CC は結果の構造化・保存に専任する。

```
YouTube URL → Gemini API（視聴+分析）→ プロンプト仕様書 → CC が保存
```

## 出力: 3モダリティのプロンプト仕様書

- **音楽**: Style / Tags / Lyrics 制御タグ（Suno/Udio/MiniMax 用）
- **画像**: ジャケット・ビジュアル用プロンプト
- **動画**: MV・ショート動画用

## analyze-song との違い（重要）

| | reverse-engineer-song | analyze-song |
|---|---|---|
| 分析手法 | 定性（Gemini が言語記述） | 定量（数値抽出） |
| 出力 | プロンプト仕様書 | features.json |
| 目的 | **再現** | **客観比較** |
| 入力 | YouTube URL | 音源（YouTube/MP3） |

両者は**完全独立**。定性で「再現の型」を、定量で「客観データ」を得て補完し合う。

## 連携

- 分析結果 → make-song の作曲参照（「この型で作りたい」）
- 音楽プロンプトは本スキル、映像プロンプトは video-prompt-spec（役割分離）
