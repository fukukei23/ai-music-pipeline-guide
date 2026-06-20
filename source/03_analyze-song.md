# 03 analyze-song — 楽曲定量分析

> 音源から BPM/キー/コード進行/メロディ輪郭/音域/phrase_repetition を数値抽出し、features.json + 楽譜PNG/PDF + report.md を出力。定性（reverse-engineer-song）とは完全独立の定量分析専用。

---

## できること

音源（YouTube URL / ローカル MP3）から以下を**数値**で抽出:

- **BPM**・テンポ信頼度（librosa）
- **キー**・スケール・信頼度（music21）
- **コード進行**（music21 chordify）
- **メロディ音域**（music21）
- **phrase_repetition**: 前半/後半の音程同一性検出
- **structure**: 曲構成

## トリガー

「楽曲分析して」「曲を定量分析」「BPM/コード抽出」「名曲っぽさ分析」「/analyze-song」

## 分析パイプライン（Phase 1a）

```
音源(yt-dlp/ローカル)
 → librosa(BPM/duration)
 → basic_pitch(MP3→MIDI・onnxruntime)
 → music21(キー/コード/メロディ/phrase_repetition/structure)
 → MuseScore(楽譜PNG/PDF)
 → features.json + report.md
```

7モジュール構成・CLI で一括実行。

## 出力（`analysis/` 配下）

- `features.json` — 全特徴量（機械用）
- `score/full-1.png` `score/full.pdf` — 五線譜（MuseScore環境依存）
- `report.md` — サマリ＋工程ログ（人間用）

## Phase 1a の既知制限（改善予定）

実測で判明した3つの制限。Phase 1b（Demucs音源分離導入）で改善対象。

> ⚠ **BPM 精度**: librosa 推定限界で、生成指定BPMと乖離する場合あり（例: 指定85→推定112・4/3倍誤差）。

> ⚠ **phrase_repetition 精度**: basic_pitch 生MIDIのノイズ（ボーカル+バック+ノイズ混入）で一致率が低くなる。メロディ単離（Phase1b Demucs）で改善。

> ⚠ **楽譜PNG**: MuseScore AppImage が WSL 環境依存で segfault する場合あり（features.json は正常生成）。

## ロードマップ

- **1a** ✅ 音源取得+分析エンジン（librosa/basic_pitch/music21・Demucs無し）
- **1b** Demucs音源分離で精度UP
- **2** 歴史的名曲特徴量DB構築
- **3** 照合エンジン + make-song 連携（「名曲っぽさ」判定）

## 連携

- 定量データ → make-song の作曲参照（Phase 3・名曲DB）
- 定性は reverse-engineer-song で（役割分離）
