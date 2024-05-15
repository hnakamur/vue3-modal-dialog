# vue3-modal-dialog

これは[Vue3でモーダルダイアログの起動をいい感じに実装する | フューチャー技術ブログ](https://future-architect.github.io/articles/20240515a/)の記事のサンプルをベースにフォーカス・トラップの実装を追加したものです。

元のコードのライセンスについては[public license でいいかと思ってます。じゃんじゃんコピーしてください](https://twitter.com/shibu_jp/status/1790616809957478875)とコメントいただいています。


フォーカス・トラップの実装は以下のコードやページを参考にしました。

* https://github.com/KittyGiraudel/focusable-selectors/blob/799829e3b8c329d679b3b414b5dfcfa257a817cf/index.js
* https://github.com/focus-trap/tabbable/blob/baa8c3044fe0a8fd8c0826f4a3e284872e1467a5/src/index.js#L1-L13
* https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/dialog.html

フォーカス・トラップについては[Allow modal dialogs to trap focus, avoiding tabbing to the URL bar · Issue #8339 · whatwg/html](https://github.com/whatwg/html/issues/8339)では不要という意見が多いようですが、[Webアプリケーションアクセシビリティ ――今日から始める現場からの改善：書籍案内｜技術評論社](https://gihyo.jp/book/2023/978-4-297-13366-5)では実装するのがよいと書かれています。

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
