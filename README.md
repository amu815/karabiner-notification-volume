# karabiner-notification-volume

macOSで **通知音(アラート音量)だけ** を `fn+F11` / `fn+F12` でキーボードから増減する [Karabiner-Elements](https://karabiner-elements.pqrs.org/) 設定。メインの出力音量(音楽・動画など)とは独立に効きます。

- `fn+F12` → 通知音 **上げ**
- `fn+F11` → 通知音 **下げ**(最後は無音=ミュート)
- 素の `F11` / `F12` は今まで通り **メイン音量**

## 仕組み

通知音の正体は macOS の **Alert volume(警告音量)** で、出力音量とは独立。AppleScript で操作できます。

```bash
osascript -e 'get volume settings'          # output / input / alert / mute を表示
osascript -e 'set volume alert volume 30'   # 通知音量だけ 0〜100 で設定
```

## なぜ `Control` ではなく `fn` なのか(ハマりどころ)

Karabiner は生の音量/メディアキー(`consumer_key_code: volume_increment` / `volume_decrement`)を **`from`(変換元)に指定しても発火しません**。EventViewer には表示されるのに掴めない、という既知の制約です。

- [#1059 Cannot remap volume keys](https://github.com/pqrs-org/Karabiner-Elements/issues/1059)
- [#1951 Can't remap volume keys](https://github.com/pqrs-org/Karabiner-Elements/issues/1951)
- [#4371 works in EventViewer but key remapping does not apply on macOS Tahoe](https://github.com/pqrs-org/Karabiner-Elements/issues/4371)

回避策として、**`fn+F11/F12` は“本物の `key_code: f11` / `f12`”を出す**点を利用します。通常キーなので Karabiner が確実に掴めます。素の `F11/F12`(fnなし)はメディアキーのままなので、メイン音量との棲み分けも自動で成立します。

## 使い方

1. [`alert-volume.json`](./alert-volume.json) を次の場所に置く:
   `~/.config/karabiner/assets/complex_modifications/alert-volume.json`
2. Karabiner-Elements → **Complex Modifications** → **Add rule** → 「通知音(アラート音量)を fn+F11/F12 で増減」を **Enable**
3. 動作確認(耳ではなく数値で):
   ```bash
   osascript -e 'set volume alert volume 20'   # 基準化
   # fn を押しながら F12 を数回
   osascript -e 'get volume settings'           # alert volume が上がり output は不変なら成功
   ```

## Claude Code に丸投げする場合

[`prompt.md`](./prompt.md) をそのまま [Claude Code](https://claude.com/claude-code) に貼れば、別の Mac でも一発で再現できます(今回ハマった前提を最初から織り込み済み)。

## 注意点

- `fn+F11` は macOS 標準だと「デスクトップを表示」。このルールが上書きします。
- 「`Control`+音量キー」にしたい場合は Karabiner では不可。`Hammerspoon`(`systemDefined` の eventtap でメディアキー＋修飾を捕捉)なら実装できます。
- `shell_command` の `osascript` は環境差を避けるため **フルパス `/usr/bin/osascript`** で記述。
- 刻みは `±10`。`shell_command` 内の `+ 10` / `- 10` を変えれば調整可能。末尾の `-e 'beep'` は変更後の音量で1回鳴らすフィードバック(不要なら削除)。

## 動作確認環境

macOS 26.5 (Tahoe) / Karabiner-Elements 16.0.0 / Apple Silicon (M2 Max)

## License

[MIT](./LICENSE)
