# fizz-reading-corrector

Fizz の **読み補正フィルタ**(TTS 前段)。TTS に渡す直前のテキストで、固有名詞や
誤読しやすい語の「漢字→かな」置換をかける(例: 「空澄」→「そらすみ」)。

LLM 出力に依存せず読みを固定するための保険。persona に読みを書いても LLM が
たまに漢字単独で出すケースを、ここで一手にカバーする。

## I/O

stdin: 1 行 1 テキスト(プレーン)→ stdout: 補正後テキスト。

```sh
echo '空澄ちゃん、配信好き?' | fizz-reading-corrector
# → そらすみちゃん、配信すき?
```

ルールは順序を持つ。「空澄(そらすみ)」のような括弧付き読みを先に潰してから裸の
「空澄」を置換する(逆順だと「そらすみ(そらすみ)」と二重化する)。

## 辞書の差し替え

環境変数 `FIZZ_READING_DICT` に TSV パスを与えると、ルールをそれに差し替える:

```
空澄	そらすみ          # リテラル全置換
re:空\s*澄	そらすみ      # 正規表現置換 (re: プレフィックス)
```

省略時は lime(ラムネ)の既定ルール。

## 開発

```sh
almide check src/main.almd
almide test
almide build src/main.almd -o build/fizz-reading-corrector
```

ツールチェーン: [almide](https://github.com/almide/almide) v0.26.6+。依存なし
(純テキスト処理)。

## §5 (音声系) のスコープについて

§5 のうち Almide に移植できるのはこの読み補正(純テキスト)まで。TTS 合成
(Aivis/VOICEVOX)はバイナリ音声の取得が必要で、Almide の HTTP クライアントが
String 専用(bytes 非対応)のため移せない。lipsync / mic / audio-unlock / 再生は
ブラウザの Web Audio API 依存で、native/WASM CLI には存在しない。これらは
openaituber 本体(TS / Web Audio)に残す。
