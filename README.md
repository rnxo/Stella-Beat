<p align="center">
  <img src="docs/stella-beat-logo.png" alt="Stella Beat" width="480">
</p>

# Stella Beat

Unity 製の 5 レーン鍵盤型リズムゲームです。JSON 譜面に沿って流れてくるノーツを `D` `F` `Space` `J` `K` で叩き、判定・コンボ・スコア（100 万点満点）とランクを競います。

## デモ

実際のプレイ映像です。

https://github.com/user-attachments/assets/95fad1d1-256a-45af-9913-f4ebf15ee156

## リポジトリ構成

このリポジトリは 2 つの Unity プロジェクトが入れ子になっています。

| パス | 内容 | Unity バージョン |
| --- | --- | --- |
| `./`（ルート） | ラッパー用の空プロジェクト（`SampleScene` のみ） | 2022.3.28f1 |
| `Assets/rhythm/` | **ゲーム本体**（旧名 `rhythm game`） | 2021.3.24f1 |

`Assets/rhythm` は別リポジトリ（<https://github.com/rnxo/Stella-Beat.git>）が gitlink として記録されていますが `.gitmodules` は未登録です。ルートを clone しただけでは中身が取得できないため、別途取得してください。

```sh
git clone https://github.com/rnxo/Stella-Beat.git "Assets/rhythm"
```

## 動かし方

1. Unity Hub に `Assets/rhythm` をプロジェクトとして追加する（Unity 2021.3.24f1 推奨）
2. `Assets/Resources/Scenes/select.unity` を開いて再生する
3. `Assets/rhythm/build/` にビルド済みの成果物（Windows 版 `rhythm game.exe` / WebGL 版 `index.html`）もあります

## 遊び方

- **選曲画面（`select`）**: 左右で曲を切り替え、難易度ボタンで Easy / Normal / Hard を切り替えます
- **設定画面（`Speed`）**: ノーツ速度を `+` / `-` で調整します（初期値 0 のため、プレイ前に設定が必要です）
- **プレイ画面（`game`）**: `Enter` で楽曲スタート。レーンは左から `D` `F` `Space` `J` `K`
- **リザルト画面（`Result`）**: スコア・コンボ・判定内訳とランクを表示します

### 判定とスコア

| 判定 | 許容誤差 | 配点 |
| --- | --- | --- |
| Perfect | ±0.08 秒 | 5 |
| Great | ±0.12 秒 | 4 |
| Good | ±0.16 秒 | 3 |
| Miss | 0.2 秒超過 | 0 |

スコアは `獲得点 ÷ (ノーツ数 × 5)` を 1,000,000 点満点に換算した値です。ランクは Great + Good の合計数で決まります（0 なら最高ランク）。

## 譜面データ

譜面は `Assets/rhythm/Assets/Resources/*.json`、楽曲は `Assets/rhythm/Assets/Resources/Musics/*.wav` に、`<曲略称> <難易度>`（例: `Ao Easy`）という名前で対になって配置されています。

```json
{
  "name": "Ao to natsu Easy",
  "maxBlock": 5,
  "BPM": 185,
  "offset": 0,
  "notes": [
    { "LPB": 4, "num": 16, "block": 3, "type": 2,
      "notes": [ { "LPB": 4, "num": 32, "block": 3, "type": 2, "notes": [] } ] }
  ]
}
```

- `LPB` / `num`: 拍の分解能と位置（`60 / BPM × num / LPB` 秒に配置）
- `block`: レーン番号（0〜4）
- `type`: `1` = 単ノーツ、`2` = ロングノーツ（`notes` に終端ノーツをネスト）

## 主なスクリプト

`Assets/rhythm/Assets/Resources/Scripts/` 以下にあります。

| ファイル | 役割 |
| --- | --- |
| `GManager.cs` | シーンをまたぐ共有状態（曲 ID・速度・スコア・判定数）のシングルトン |
| `IManager.cs` | 選曲画面のジャケット送りと、楽曲 / 譜面パスの決定 |
| `NotesManager.cs` | 譜面 JSON の読み込みとノーツ / ロングノーツの生成 |
| `Notes.cs` / `LongNotes.cs` / `LongLastNotes.cs` | ノーツの移動処理 |
| `Judge.cs` / `NestedJudge.cs` / `AvgPositionJudge.cs` | 単ノーツ・ロングノーツ終端・ロングノーツ本体の判定 |
| `MusicManager.cs` | `Enter` での楽曲再生と開始時刻の記録 |
| `RManager.cs` | リザルト表示とランク判定、スコアのリセット |
| `Speed.cs` / `Difficulity.cs` / `Changer.cs` / `Light.cs` | ノーツ速度設定 / 難易度表示 / シーン遷移 / レーン発光 |
