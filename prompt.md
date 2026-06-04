# Claude Code セットアッププロンプト

以下をそのまま [Claude Code](https://claude.com/claude-code) に貼り付けてください。別の Mac でも同じ設定を再現できます。落とし穴(修飾は mandatory にする / キーボードによってF行の出すコードが違う)を最初から織り込んであります。

````text
macOSで「通知音(alert volume)」だけを ⌃(左Control)+F11 / ⌃+F12 で下げ/上げできるように、
Karabiner-Elements を設定してください。メインの出力音量とは独立に効くようにします。
素の F11/F12 は従来どおりメイン音量のまま残すこと。

## 前提と落とし穴(先に押さえること)
- 通知音の実体は macOS の Alert volume で、出力音量とは独立。
  操作は osascript -e 'set volume alert volume N'(N=0〜100)。
- まず Karabiner の EventViewer で「F12 を押すと何が出るか」を確認:
  - key_code: f12 が出る → このプロンプトの方式(key_code + mandatory修飾)でOK。
  - consumer_key_code: volume_increment が出る → Karabiner はメディアキーを from に
    できない既知制約(issue #1059 / #1951、Tahoe は #4371)。Karabiner では不可なので、
    その旨を伝えて Hammerspoon 方式(systemDefined eventtap)を提案して中止する。
- 【重要】from の modifiers は必ず mandatory:["left_control"] にする。
  optional:["any"] だけにすると、修飾なしの素の F11/F12 まで横取りしてしまい、
  通常のメイン音量調整が壊れる。
- shell_command の osascript は フルパス /usr/bin/osascript で書くこと。

## やること
1. 次のファイルを作成(既存ルールには一切触れない・追加するだけ):
   ~/.config/karabiner/assets/complex_modifications/alert-volume.json

{
  "title": "通知音(アラート音量)を ⌃+F11/F12 で増減",
  "rules": [
    {
      "description": "⌃(Left Control)+F11/F12 で alert volume(通知音)を増減。素の F11/F12 は通常のメイン音量のまま。",
      "manipulators": [
        {
          "from": { "key_code": "f11", "modifiers": { "mandatory": ["left_control"], "optional": ["any"] } },
          "to": [{ "shell_command": "/usr/bin/osascript -e 'set v to (alert volume of (get volume settings)) - 10' -e 'if v < 0 then set v to 0' -e 'set volume alert volume v' -e 'beep'" }],
          "type": "basic"
        },
        {
          "from": { "key_code": "f12", "modifiers": { "mandatory": ["left_control"], "optional": ["any"] } },
          "to": [{ "shell_command": "/usr/bin/osascript -e 'set v to (alert volume of (get volume settings)) + 10' -e 'if v > 100 then set v to 100' -e 'set volume alert volume v' -e 'beep'" }],
          "type": "basic"
        }
      ]
    }
  ]
}

2. python3 -m json.tool で JSON 妥当性を確認。
3. 有効化: ユーザーに「Karabiner-Elements → Complex Modifications → Add rule →
   『通知音(アラート音量)を ⌃+F11/F12 で増減』を Enable」と案内する。
   (CLI派なら karabiner.json の selected:true プロファイルの complex_modifications.rules
    先頭に上記 rule をマージし、既存ルールは保持したまま
    karabiner_cli --select-profile "$(karabiner_cli --show-current-profile-name)" で再読込)
4. 動作確認(耳でなく数値で。あなたはキーを押せないのでユーザーに押してもらう):
   - osascript -e 'set volume output volume 30' と 'set volume alert volume 20' で基準化
   - ユーザーに「素の F12 を2回」→ output が上がる(メイン音量は従来どおり)
   - ユーザーに「左Control を押しながら F12 を2回」→ alert が上がる(通知音だけ)
   - osascript -e 'get volume settings' で両方を確認
5. 補足: 左Control を使う。別の修飾キーが良ければ mandatory の値を変える。
````
