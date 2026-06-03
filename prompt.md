# Claude Code セットアッププロンプト

以下をそのまま [Claude Code](https://claude.com/claude-code) に貼り付けてください。別の Mac でも一発で同じ設定を再現できます。「音量キーは `from` にできない」という今回ハマった前提を最初から織り込んであるので、試行錯誤をスキップできます。

````text
macOSで「通知音(alert volume)」だけを fn+F11 / fn+F12 で下げ/上げできるように、
Karabiner-Elements を設定してください。メインの出力音量とは独立に効くようにします。

## 重要(先に知っておくべき罠。これで時間を無駄にしないこと)
- Karabiner は生の音量/メディアキー(consumer_key_code: volume_increment /
  volume_decrement)を from(変換元)に指定しても発火しない。EventViewer には出るのに
  掴めない既知の制約(GitHub issue #1059 / #1951、macOS Tahoe では #4371)。
  → consumer_key_code を from に使う方式は絶対に試さないこと。動きません。
- 代わりに、fn+F11 / fn+F12 は“本物の key_code f11 / f12”を出すので、通常キーとして
  Karabiner が確実に掴めます。これを使ってください。素の F11/F12 はメイン音量のまま。
- 通知音量の実体は macOS の Alert volume で、出力音量とは独立。
  操作は osascript -e 'set volume alert volume N'(N=0〜100)。
- shell_command の osascript は フルパス /usr/bin/osascript で書くこと。

## やること
1. 次のファイルを作成(既存ルールには一切触れない・追加するだけ):
   ~/.config/karabiner/assets/complex_modifications/alert-volume.json

{
  "title": "通知音(アラート音量)を fn+F11/F12 で増減",
  "rules": [
    {
      "description": "fn+F11/F12 で alert volume(通知音)を増減。素の F11/F12 は通常のメイン音量のまま。",
      "manipulators": [
        {
          "from": { "key_code": "f11", "modifiers": { "optional": ["any"] } },
          "to": [{ "shell_command": "/usr/bin/osascript -e 'set v to (alert volume of (get volume settings)) - 10' -e 'if v < 0 then set v to 0' -e 'set volume alert volume v' -e 'beep'" }],
          "type": "basic"
        },
        {
          "from": { "key_code": "f12", "modifiers": { "optional": ["any"] } },
          "to": [{ "shell_command": "/usr/bin/osascript -e 'set v to (alert volume of (get volume settings)) + 10' -e 'if v > 100 then set v to 100' -e 'set volume alert volume v' -e 'beep'" }],
          "type": "basic"
        }
      ]
    }
  ]
}

2. python3 -m json.tool で JSON 妥当性を確認。
3. 有効化: ユーザーに「Karabiner-Elements → Complex Modifications → Add rule →
   『通知音(アラート音量)を fn+F11/F12 で増減』を Enable」と案内する。
   (CLI派なら karabiner.json の selected:true プロファイルの complex_modifications.rules
    先頭に上記 rule をマージし、既存ルールは保持したまま
    karabiner_cli --select-profile "$(karabiner_cli --show-current-profile-name)" で再読込)
4. 動作確認(耳でなく数値で。あなたはキーを押せないのでユーザーに押してもらう):
   - osascript -e 'set volume alert volume 20' で基準化(現在値は控えて後で戻す)
   - ユーザーに「fn を押しながら F12 を2回」押してもらう
   - osascript -e 'get volume settings' で alert volume が上がり output は不変なら成功。
     output が上がって alert が動かない場合は、その環境で fn+F12 が f12 を出していないので
     内蔵キーボードで再確認するか、Hammerspoon 方式へ切替。
5. 補足: fn+F11 は標準だと「デスクトップを表示」。上書きする旨を伝える。
   「fn ではなく Control+音量キー」が必要なら Karabiner では不可で、Hammerspoon が要る。
````
