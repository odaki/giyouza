# ギヨウザルーレット

崎陽軒シウマイ風パッケージをモチーフにした、ブラウザで遊べるルーレットゲームです。

## 遊び方

1. ブラウザで `index.html` を開く
2. **スペースキー** または **マウス左クリック** で矢を放つ
3. 4つのルーレットに順番に矢を射て、出た文字の組み合わせを確認

## 当たり一覧

| 種別 | 組み合わせ | エフェクト |
|------|-----------|-----------|
| 🎊 大当たり | ギ ヨ ウ ザ | ギヨウザ大量落下（物理演算） |
| ⭐ 中当たり① | ギ ザ ギ ザ | ギザギザ画面エフェクト |
| ⭐ 中当たり② | ウ ヨ ウ ヨ | ウヨウヨ虫が出現 |
| ⭐ 中当たり③ | ウ ザ ウ ザ | ウザウザテキスト大量発生 |
| ⭐ 中当たり④ | ギ ウ ギ ウ | ギウギウぎゅうぎゅう詰め |
| ⭐ 中当たり⑤ | ギ ギ ギ ギ | ギギギギ火花・グリッチ |
| ⭐ 中当たり⑥ | ヨ ヨ ヨ ヨ | ヨヨヨヨ上下に弾むヨーヨー |
| ⭐ 中当たり⑦ | ウ ウ ウ ウ | ウウウウ幽霊が浮かび上がる |
| ⭐ 中当たり⑧ | ザ ザ ザ ザ | ザザザザ激しい雨 |
| ⭐ 中当たり⑨ | ギ ヨ ギ ヨ | ギョギョびっくり魚が乱舞 |

## 操作

| キー | 動作 |
|------|------|
| スペース / 左クリック | 矢を放つ・リスタート |
| G | チート: 現在の的に「ギ」を射抜く |
| Y | チート: 現在の的に「ヨ」を射抜く |
| U | チート: 現在の的に「ウ」を射抜く |
| Z | チート: 現在の的に「ザ」を射抜く |

## 技術仕様

- HTML5 Canvas 2D API のみ（依存ライブラリなし）
- 六角形最密充填による物理スタッキング（ギウギウ演出）
- 剛体円衝突シミュレーション（ギヨウザ落下・堆積・崩落）
- `index.html` 単一ファイル完結

## iOS Safari Web Audio 実装ノート

iOS Safari の Web Audio API は制約が多い。実装時に判明した仕様を記録する。

### AudioContext の状態遷移

| 状態 | 原因 | 復帰方法 |
|------|------|----------|
| `suspended` | 生成直後・通常の停止 | ユーザーアクティベーション内で `resume()` |
| `interrupted` | バックグラウンド移行後 | `close()` + `new AudioContext()` で再作成 |
| `running` | 正常動作中 | — |

### 重要ルール

**1. `audioCtx.suspend()` 禁止**  
`suspend()` 後の `resume()` が不安定。無音化は `masterGain.gain.value = 0` のみで行う。

**2. サイレントバッファは `masterGain` 経由必須**  
`ctx.destination` 直結では AVAudioSession が開通しないことがある。  
音声パス全体（BGM・SE・ジングル）をアクティブにするには `masterGain → destination` を通す必要がある。

**3. `visibilitychange` で `resume()` しない**  
バックグラウンド復帰時に `resume()` を呼ぶと `interrupted` → `suspended` に変わり、  
`interrupted` 状態の検知（AudioContext 再作成）が動かなくなる。  
フォアグラウンド復帰の音声復帰処理は、ユーザーが PAUSE 解除ボタンを押すタイミングに委ねる。

**4. `interrupted` は `resume()` では戻せない**  
`audioCtx.close()` して `new AudioContext()` を作り直す（`_recreateAudioGraph()`）。  
ユーザーアクティベーション内で `new AudioContext()` すると iOS が即 `running` にする。

**5. START ボタンの音声開始は `touchend` で行う**  
`touchstart` で `resume()` + サイレントバッファ（`_unlockAudio()`）を発行し、  
指を離す `touchend`（別のユーザーアクティベーション）で `initGame()` を呼ぶ。  
`touchstart` と同一イベント内で音声スケジューリングすると、iOS が AudioContext を  
`running` にする前にノートが積まれてしまい無音になることがある。

**6. `new AudioContext()` はユーザーアクティベーション前でも作成できる**  
`suspended` 状態で待機するだけで、`resume()` はジェスチャー内で呼べばよい。  
ページロード時に生成しておくと BGM のオフラインレンダリングを事前に完了できる。

### オーディオ関数の役割分担

| 関数 | 用途 |
|------|------|
| `_unlockAudio()` | 全タッチ共通。機会があれば unlock する（fire-and-forget） |
| `_resumeAudioCtx(cb)` | 特定の音声操作前に running を保証してから cb を呼ぶ |
| `_recreateAudioGraph()` | `interrupted` 復帰専用。AudioContext ごと再作成 |

## デバッグ

### Web Inspector の接続方法

**Mac Safari + iOS Safari（実機）**

1. iPhone: 設定 → Safari → 詳細 → Web インスペクタ をオン
2. USB ケーブルで Mac に接続
3. Mac Safari: 設定 → 詳細 → 「Web デベロッパ用の機能を表示」をオン
4. iPhone の Safari でページを開く
5. Mac Safari のメニュー: 開発 → デバイス名 → ページを選択

**Mac Safari + iOS シミュレータ（Xcode）**

1. Xcode のシミュレータを起動し、Safari でページを開く
2. Mac Safari の「開発」メニュー → Simulator → ページを選択  
   （表示されない場合は Mac Safari を再起動）

## 開発セットアップ

### pre-commitフック（バージョン自動インクリメント）

`index.html` 内の `VERSION` 定数はコミットのたびに自動でインクリメントされます。
`git clone` 後や環境を変えた際は以下のコマンドでフックを設定してください。

```sh
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
# pre-commit: index.html の VERSION 定数を自動インクリメントする
# 例: 'v3.10' → 'v3.11', 'v3.99' → 'v4.00'

FILE="index.html"

CURRENT=$(grep -o "const VERSION = 'v[0-9]*\.[0-9]*'" "$FILE" | grep -o "v[0-9]*\.[0-9]*")
if [ -z "$CURRENT" ]; then
  echo "pre-commit: VERSION定数が見つかりません。スキップします。" >&2
  exit 0
fi

MAJOR=$(echo "$CURRENT" | sed "s/v\([0-9]*\)\..*/\1/")
MINOR=$(echo "$CURRENT" | sed "s/v[0-9]*\.\([0-9]*\)/\1/")
# 10# プレフィックスで強制10進数評価（08,09 が8進数エラーになるのを防ぐ）
MINOR_DEC=$((10#$MINOR))
if [ "$MINOR_DEC" -ge 99 ]; then
  NEXT="v$((10#$MAJOR + 1)).00"
else
  NEXT="v${MAJOR}.$(printf "%02d" $((MINOR_DEC + 1)))"
fi

sed -i "" "s/const VERSION = '${CURRENT}'/const VERSION = '${NEXT}'/" "$FILE"

git add "$FILE"

echo "pre-commit: VERSION ${CURRENT} → ${NEXT}"
EOF
chmod +x .git/hooks/pre-commit
```

> **注意:** Windowsの場合、`sed -i ""` の代わりに `sed -i` を使用してください（GNU sed）。
> Git for Windowsのbashであれば上記のまま動作する場合もあります。

## ライセンス

MIT License
