---
title: "Vimでyamlを書くための環境を整える"
date: 2021-01-16T19:22:16+09:00
draft: false

# post thumb
image: "images/featured-post/post-1.png"

# meta description
description: "this is meta description"

# taxonomies
categories:
  - "Vim"
tags:
  - "Vim"
  - "Yaml"

# post type
type: "featured"
---

最近、yamlを書く機会が増えてきました。

その中で、不便な点がいくつかあったのでVimで快適にyamlを書けるように色々と設定しました。

今回は、

- [efm-langserver](https://github.com/mattn/efm-langserver)・[yaml-language-server](https://github.com/redhat-developer/yaml-language-server)を使う方法
- efm-langserverから[yamllint](https://github.com/adrienverge/yamllint)を使う方法
- AWSのテンプレートで使用する組み込み関数(!Ref・!Sub等)がエラーになるのを防ぐ方法

を紹介したいと思います。

## efm-langserverとyaml-language-serverの導入

今回は、[vim-lsp](https://github.com/prabirshrestha/vim-lsp)、[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)を使用しました。

Pluginのインストールには[vim-plug](https://github.com/junegunn/vim-plug)を使用しています。

.vimrcに以下を書きます。
```
call plug#begin()
Plug 'prabirshrestha/vim-lsp'
Plug 'mattn/vim-lsp-settings'
Plug 'prabirshrestha/asyncomplete.vim'
Plug 'prabirshrestha/asyncomplete-lsp.vim'
call plug#end()
```

以下を実行
```
:PlugInstall
```
LSPのインストールは、
```
:LspInstall yaml-language-server
```
```
:LspInstall efm-langserver
```
LSPの動作確認は、
```
:LspStatus
```
により可能です。

![image](../../images/post/post-1/1.png)

## LSPの設定

上記では、L20の!Refがエラーとして表示されているので設定を追加します。

.vimrcにLSPごとに設定を書いていきます。
```
let g:lsp_settings = {
  \  'efm-langserver': {
  \    'disabled': v:false,
  \  },
  \  'yaml-language-server': {
  \     'workspace_config': {
  \       'yaml': {
  \         'customTags': [
  \           '!Sub',
  \           '!Ref',
  \         ],
  \       },
  \     },
  \   },
```
[参考](https://github.com/redhat-developer/yaml-language-server/issues/77#issuecomment-511768680)

ここでは、efm-langserverの有効化、yaml-language-serverのcustomTagsの設定を行なっています。

customTagsの設定によりyaml-language-serverが!Subや!Ref等をエラーとして表示するのを防ぎます。

エラー表示したくないものは、customTagsに書くことでエラーになるのを防ぎます。

[customTags参考](https://github.com/redhat-developer/yaml-language-server/issues/77#issuecomment-474864515)
## efm-langserverの設定

上記の設定の追加により、efm-langserverが有効になったのですが、まだyamllintは動作していません。

yamllintが動作するよう設定を追加します。

$HOME/.config/efm-langserver/config.yamlに言語ごとの設定を書いていきます。(Windowsでは%APPDATA%\efm-langserver\config.yamlになります。)

以下が、今回使用したconfig.yamlです。
```
tools:
  yaml-yamllint: &yaml-yamllint
    lint-command: 'yamllint -f parsable -'
    lint-stdin: true
languages:
  yaml:
    - <<: *yaml-yamllint
```

## 最後に

Vim上でyamllint、yaml-language-serverを使えるようなり、yamlを快適に書けそうです。

![image](../../images/post/post-1/2.png)
