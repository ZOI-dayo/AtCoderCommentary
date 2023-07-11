---
title: AtCoder用のDiscord Botを作った話
tags:
  - AtCoder
  - TypeScript
  - 競技プログラミング
  - discord
  - Deno
private: false
updated_at: '2023-05-29T15:55:30+09:00'
id: b9fbd1b50eadbf855f34
organization_url_name: null
---
## これは何

https://github.com/ZOI-dayo/AtCoder_Discord_Bot

AtCoderの色々をしてくれるBotです

[maguro氏の記事](https://festival2023.npca.jp/bushi/maguro/)を見て、自分でも作ってみたかったのでDenoの練習代わりに作ってみました
(~~パクリです~~強い影響を受けています)

## 機能

- 当日のコンテストを通知(朝7時)
- コンテスト結果が出たとき、(設定したユーザーの)レート変動を通知
- 学校のAJLの順位変動を通知

AJL関係を入れている辺りからお察しかもしれませんが、「学校のPC部のDiscordサーバー」に導入することを想定して作ってます

機能が少ないので、なにか案があればコメントかissueかプルリクください

## 使い方

1. denoをセットアップ
1. Discord API Tokenを作っておく
1. [GitHub]をクローン
1. プロジェクト直下に.env.localファイルを作成し、`DISCORD_TOKEN=〇〇〇〇`と記述
1. `$ deno task start`

([僕の自鯖で動いているBotインスタンス](https://discord.com/api/oauth2/authorize?client_id=1105087451426463805&permissions=3072&scope=bot)もいるのですが、デバッグしてるときにバグって荒らしたり、いきなりサ終するかもなので、各々でホストすることをおすすめします)

## コマンド

設定などは、Discordのスラッシュコマンドで行うようにしています

- /atcoder set school {学校名} {中学/高校}
AJL用に学校を登録します
学校名は正式名称でお願いします(AJLのページにあるもの)
中高は手動選択です

- /atcoder set channel
Botがいろいろ送信するチャンネルを設定します
コマンドを打ったチャンネルが送信先として保存されます

- /atcoder register {AtCoderユーザー名}
これを実行すれば、以降のコンテスト結果通知で自分の結果が表示されるようになります

## 開発について

### 言語

AtCoderの色々をやってくれるBotなので、どうしてもスクレイピングやJSONからは逃げられそうにない...と思い、JS周辺から選ぶことにしました
で、DiscordのBotであってWebページでないので素のJavaScriptは使えないです
となるとNode.jsか? と思いましたが、せっかくなので使ったことのないDenoを触ってみることにしました

### Discord関連

discordenoというライブラリがあったのでそのまま利用しました
DiscordのBotは昔ちょっとだけ作ろうとしたことがあって、「Guildってサーバーのことだよな〜?」ぐらいの知識だったんですが、まあなんとかなりました

### スクレイピング

まず、AtCoderの一部のページは、URL末尾に「/json」をつけるとJSON形式でデータがもらえます
(ただ最近のDDoS騒動の影響で一部のページはログインしないと見れない+アクセス過多だと止められるので注意)
たとえば<https://atcoder.jp/contests/abc300/results/json>などです
これは`await (await fetch('https://atcoder.jp/contests/abc300/results/json')).json();`でOKです

そうでないもの(例えばコンテスト予定やAJL関係)は普通に`fetch('hoge').text()`を`new DOMParser().parseFromString(text, "text/html");`に放り込んでDOM操作です
そういえば、AJLの結果って<https://img.atcoder.jp/ajl2023/output_high_school.html>で見れるんですが、どうしてimg.atcoder.jpに配置されてるんでしょう...?謎

### データ保存

「どのチャンネルに送信するか」とか、「どの学校か」とかはBotが停止しても保存されててほしいですね
なので、まあ普通にJSONに書けばいいんですが、あまり使ったことなかったのでSQLiteに入れています
sqliteとかいうそのままな名前のライブラリがあったのでそれ入れて適当実装です

### 時間管理

例えば「コンテストがある日の朝7時に関数を実行」というやつです
日付指定で実行、という便利な関数が見つからなかった(あるのかもしれませんが)ので、`setInterval(() => {}, (予定時刻[ms]) - (現在時刻[ms]))`でなんとかしました

ただ、「コンテストのある日の朝7時」を取得するときに、「指定時間以前で最も近い日本時間0:00」を見つけ出す方法がよくわからなかった(date_fnsにstartOfDayという関数はあるのですが、世界標準時の0:00を返してきた)ので、DateTimeライブラリを自作しました

ゴミコードですが、まあ今回の用途では十分でした

### 開発環境

Denoか〜、Web系技術だからVSCodeが得意そうだな〜、と思い、これまた使ったことのなかったGitHub Codespacesを使ってみました

普通のFreeプランでは120CPU時間/月しかないので、最低ランクの2Coreマシンでも60時間/月しか使えないのがちょっと不安ですが、使用感はかなり良かったです
また小さなプロジェクトをやることになったら使ってみようと思います

## 感想

最初思ってたよりは結構コードを書く羽目になりました...ここら辺の見積もりをちゃんとできるようになりたいですね
いつもBotを作る時は途中で飽きてやめてしまうので、動作まで持って来れて良かったです
