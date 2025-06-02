# Mac と Linux でシステムフォントを同じにしよう

フォントの字形は、可読性や心理状態に影響を与え、作業へのモチベーションや集中力にも関係すると私自身は思います。

たとえば、**ゴシック体 (Sans Serif)** は直線的で均一な線幅が特徴で、モダンで堅実な印象を与えます。視認性が高く、コードエディタやターミナルのように情報をすばやく読み取る必要がある場面や公式文書に適していると言われています。

一方、**明朝体 (Serif)** は筆の流れを感じさせるセリフや線の強弱が特徴で、柔らかくエレガントな印象を与えます。テキストエディタやドキュメント作成時には、落ち着いた雰囲気をもたらし、文章執筆やアイデア出しといったクリエイティブな作業や内部文書に適していると言われています。ただし、細い線が多く、小さな文字サイズでは視認性が低下するため、コードやログの確認には不向きなことがあります。

また、ホラーやサスペンス作品で見られるような歪んだ字形や崩れたフォントは、恐怖や不安を強調する効果があります。特定の演出には効果的ですが、日常の作業環境では視覚的な違和感やストレスの原因となることがあります。

このように、フォントの字形が与える印象は視覚的な快適さに直結します。同じ「ゴシック体」でもフォントごとに形が異なり、さらに Mac と Linux ではレンダリングエンジンやフォント選択の違いにより、印象が変わってしまいます。幸い Apple は macOS のシステムフォント「SF Pro」を公式に公開しており、これを Linux 側にも導入することで、両環境での視覚体験の差を軽減できます。

## ⚠️ 注意事項 ⚠️

Linux では Fontconfig によってフォント設定を細かく制御できますが、**アプリケーション側がそれを参照しない**場合があります。

たとえば、以下のような文字:

```text
⚠ (U+26A0) + <U+FE0F> = ⚠️
```

この文字は Unicode の Variation Selector-16 (U+FE0F) を使って、テキストスタイルの ⚠ を絵文字スタイルの ⚠️ に切り替える仕組みになっています。

この「Base + Variation Selector」による絵文字表示は、私の確認した限りでは以下のようにアプリごとで挙動が異なります:

| Apps               | Display | Result |
| ------------------ | ------- | ------ |
| VS Code            | ⚠️      | ✅     |
| Firefox / Chromium | ⚠️      | ✅     |
| gnome-terminal     | ⚠️      | ✅     |
| Obsidian           | ⚠       | ❌     |
| Alacritty          | ⚠       | ❌     |

Alacritty のような GPU アクセラレーションに特化したターミナルエミュレータでは、libfontconfig を経由せずに直接フォントを処理したり、フォールバックレンダリングを簡略化しているため、意図通りに表示されないことがあります。

Obsidian は Electron ベースのアプリで、内部に組み込まれた Chromium のレンダリングロジックや CSS 指定によって、OS 側のフォント設定が反映されにくくなることがあります。

そのため、たとえ Fontconfig で SF Pro や Noto Color Emoji を優先するように設定していても、アプリケーション独自のレンダラやフォント指定によって、意図と異なる表示になることがあります。しかし、半角文字は無関係なので視覚体験には軽微な影響と思われます。

## 📋 前提環境

以下のフォントがインストールされているものとします。

- `Noto Sans CJK JP`
- `Noto Serif CJK JP`
- `Noto Sans Mono CJK JP`
- `Noto Color Emoji`
- `MesloLGS NF (任意)`

Noto 系フォントのインストール例:

```sh
sudo apt install fonts-noto-cjk fonts-noto-color-emoji
```

## 📜 要件

- sans-serif の半角文字: SF Pro Text
- sans-serif の全角文字: Noto Sans CJK JP
- monospace の半角英数: SF Mono
- monospace の全角文字: Noto Sans Mono CJK JP
- serif: Noto Serif CJK JP
- emoji: Noto Color Emoji
- Arial / Helvetica / Monaco など well known フォントファミリは sans-serif / monospace に代替
- Mac はリガチャが有効なので Linux でも有効にする
- Powerline は MesloLGS NF を利用する (任意)

SF Pro ではなく SF Pro Text を選択するのは、文字間隔が適切だからです。SF Pro は Fontconfig 環境では狭すぎと感じるため。

## ⚒ インストール手順

1. [Apple Developer サイト](https://developer.apple.com) から `SF-Pro.dmg` をダウンロード
2. フォントファイルを展開・インストール:

   ```sh
   7z x SF-Pro.dmg
   cd SFProFonts
   7z x SF\ Pro\ Fonts.pkg
   7z x Payload\~
   cp Library/Fonts/* ~/.local/share/fonts/
   ```

3. Fontconfig 設定ディレクトリの作成:

   ```sh
   mkdir -p ~/.config/fontconfig/conf.d
   ```

4. カスタム設定ファイルの配置:

   ```sh
   cp 99-user.conf ~/.config/fontconfig/conf.d/
   ```

5. フォントキャッシュの更新:

   ```sh
   fc-cache -fv
   ```

6. GNOME セッション再起動 (**Wayland 環境ではログアウト**):

   ```sh
   gnome-shell --replace
   ```

## ⚙ Gnome Tweaks 設定例

`gnome-tweaks` の Fonts 項目を以下のように調整します。

- Prefferd Fonts
  - Interface Text: Sans 10pt
  - Document Text: Sans 10pt
  - Monospace Text: Monospace 11pt
- Rendering
  - Hinting: None
  - Antialiasing: Subpixel (for LCD screens)

## 📚 参考資料

- [Fontconfig ドキュメント](https://www.freedesktop.org/wiki/Software/fontconfig/)
- [ArckWiki: フォント設定](https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88%E8%A8%AD%E5%AE%9A)

## ⚖ ライセンス

この設定ファイルは自由に使用、改変、再配布できます。
