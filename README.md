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

## デバッグ

### デバッグオーバーレイ（スナップショットログ）

PAUSE/UNPAUSE 時などの音声状態をスナップショットとしてログに記録します。  
Safari の Web Inspector コンソールから以下の関数で表示/非表示を切り替えられます（デフォルト非表示）。

```js
_dbg()   // 画面左下の緑テキストオーバーレイ（スナップショットログ）をトグル
_lat()   // PAUSE中の右下レイテンシ診断・左下フレームグラフをトグル
```

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

### ログの見方

各スナップはイベント名・AudioContext 状態・BGM ソースの有無・Gain 値・BGM 積算位置を記録します。

| フィールド | 内容 |
|-----------|------|
| `ctx:` | AudioContext の state (`running` / `suspended` / `interrupted`) |
| `src:` | BGM BufferSourceNode の有無 (`SET` / `null`) |
| `mG:` | masterGain の gain 値（0=ミュート, 1=通常） |
| `bG:` | bgmGain の gain 値（通常は常に 1） |
| `wall:` | BGM 積算の最終壁時計（null なら BGM 未再生） |
| `pos:` | PAUSE 時に保存した BGM 再生位置（秒） |

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
if [ "$MINOR" -ge 99 ]; then
  NEXT="v$((MAJOR + 1)).00"
else
  NEXT="v${MAJOR}.$(printf "%02d" $((MINOR + 1)))"
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
