# karabiner-notification-volume

macOSで **通知音(アラート音量)だけ** を `⌃(Control)+F11` / `⌃+F12` でキーボードから増減する [Karabiner-Elements](https://karabiner-elements.pqrs.org/) 設定。メインの出力音量(音楽・動画など)とは独立に効きます。

- `⌃+F12` → 通知音 **上げ**
- `⌃+F11` → 通知音 **下げ**(最後は無音=ミュート)
- 素の `F11` / `F12` は今まで通り **メイン音量**

## 仕組み

通知音の正体は macOS の **Alert volume(警告音量)** で、出力音量とは独立(出力音量に対して相対)。AppleScript で操作できます。

```bash
osascript -e 'get volume settings'          # output / input / alert / mute を表示
osascript -e 'set volume alert volume 30'   # 通知音量だけ 0〜100 で設定
```

これを Karabiner で `⌃+F11/F12` に割り当てる。

## 使い方

1. [`alert-volume.json`](./alert-volume.json) を次の場所に置く:
   `~/.config/karabiner/assets/complex_modifications/alert-volume.json`
2. Karabiner-Elements → **Complex Modifications** → **Add rule** → 「通知音(アラート音量)を ⌃+F11/F12 で増減」を **Enable**
3. 動作確認(耳ではなく数値で):
   ```bash
   osascript -e 'set volume output volume 30'   # メインの基準
   osascript -e 'set volume alert volume 20'    # 通知音の基準
   # 素の F12 を数回      → output が上がる(メイン音量は従来どおり)
   # ⌃ を押しながら F12 を数回 → alert が上がる(通知音だけ)
   osascript -e 'get volume settings'
   ```

## 落とし穴(重要)

### 1. 修飾キーは `mandatory` にする

`key_code: f11/f12` を `optional:["any"]` **だけ**で捕まえると、**修飾なしの素の F11/F12 まで一致**してしまい、**通常のメイン音量調整が効かなくなる**。必ず `mandatory:["left_control"]` を付けること。そうすれば素の F11/F12 は Karabiner をスルーして macOS の音量調整に流れる。

### 2. F11/F12 が出すコードはキーボード次第

この設定が成立するのは、お使いのキーボードの F 行が **“本物の `key_code: f11/f12`”** を出している場合。Karabiner 付属の **EventViewer** で F12 を押して確認できる:

- **`key_code: f12` が出る** → 本リポジトリの通り `⌃+F11/F12` でOK。
- **`consumer_key_code: volume_increment` が出る** → これは**メディアキー**。Karabiner は音量/メディアキーを **`from`(変換元)にできない**既知の制約があり([#1059](https://github.com/pqrs-org/Karabiner-Elements/issues/1059) / [#1951](https://github.com/pqrs-org/Karabiner-Elements/issues/1951)、macOS Tahoe では [#4371](https://github.com/pqrs-org/Karabiner-Elements/issues/4371))、この場合は Karabiner では実現できない。`Hammerspoon`(`systemDefined` の eventtap でメディアキー＋修飾を捕捉)が必要。

> 同じ Mac でも内蔵キーボードと外付けで挙動が違うことがある(Keychron 等は Mac/Win トグルや fn-lock で F 行の挙動が変わる)。

## Claude Code に丸投げする場合

[`prompt.md`](./prompt.md) をそのまま [Claude Code](https://claude.com/claude-code) に貼れば、別の Mac でも一発で再現できます(上記の落とし穴を最初から織り込み済み)。

## 注意点

- `⌃+F11` を使うので **左 Control** を使用。
- `shell_command` の `osascript` は環境差を避けるため **フルパス `/usr/bin/osascript`**。
- 刻みは `±10`。`shell_command` 内の `+ 10` / `- 10` を変えれば調整可能。末尾の `-e 'beep'` は変更後の音量で1回鳴らすフィードバック(不要なら削除)。

## 動作確認環境

macOS 26.5 (Tahoe) / Karabiner-Elements 16.0.0 / Apple Silicon (M2 Max)

## License

[MIT](./LICENSE)
