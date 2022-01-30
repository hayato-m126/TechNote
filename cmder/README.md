# cmder
Linuxのterminatorみたいな使い心地を手に入れる

## インストール
```
cinst cmdermini -y
```

## なぜminiか
どうせgit for windowsは入っている。別で入れて最新にアップデートしていたほうがよい

## 設定
chocolateyでc:\tools\cmderminiに入っている前提でパスは記述する

# 参考リンク
- https://syon.github.io/refills/rid/1498646/

- λを$に変更する
    - C:\tools\cmdermini\vendor\clink.luaを編集する
    - lambdaで検索する(51行あたり)
    - local lambda = λを$に変更
- 日本語による文字崩れを治す
    - GEneral->Fonts->Monospaceのチェックを外す
- 半透明処理をやめる
    - Features->Transparencyでスライダを一番右にする。
- anaconda promptを呼べるようにする
    - cmd /k "%ConEmuDir%\..\init.bat & C:\tools\miniconda3\Scripts\activate.bat" C:\tools\miniconda3
- ショートカットから起動したときのディレクトリを指定する
    - C:\tools\cmdermini\Cmder.exe %USERPROFILE%
- 右クリックから開けるようにする
    - コマンドプロンプトを管理者権限で起動する
    - cd C:\tools\cmdermini\
    - cmder.exe /REGISTER ALL
- 起動時のデフォルト端末を変更する
    - General->choose your startup taskから選ぶ