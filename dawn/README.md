<h1 align="center" style="position: relative;">
  <br>
    <img src="./assets/shoppy-x-ray.svg" alt="logo" width="200">
  <br>
  Shopify Skeleton 主题
</h1>

一个精心构建的极简 Shopify 主题，旨在帮助您快速上手。设计时考虑了模块化、可维护性和 Shopify 的最佳实践。

<p align="center">
  <a href="./LICENSE.md"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License"></a>
  <a href="./actions/workflows/ci.yml"><img alt="CI" src="https://github.com/Shopify/skeleton-theme/actions/workflows/ci.yml/badge.svg"></a>
</p>

## 快速开始

### 前置条件

在开始之前，请确保您已安装最新的 Shopify CLI：

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) — 帮助您下载、上传、预览主题并简化您的工作流程

如果您使用 VS Code：

- [Shopify Liquid VS Code 扩展](https://shopify.dev/docs/storefronts/themes/tools/shopify-liquid-vscode) — 提供专门为 Liquid 模板设计的语法高亮、代码检查、内联文档和自动补全

### 克隆

使用 Git 或 Shopify CLI 克隆此仓库：

```bash
git clone git@github.com:Shopify/skeleton-theme.git
# 或者
shopify theme init
```

### 预览

使用 Shopify CLI 预览此主题：

```bash
shopify theme dev
```

## 主题架构

```bash
.
├── assets          # 存储静态资源（CSS、JS、图片、字体等）
├── blocks          # 可复用、可嵌套、可自定义的 UI 组件
├── config          # 全局主题设置和自定义选项
├── layout          # 页面的顶层包装器（布局模板）
├── locales         # 主题国际化的翻译文件
├── sections        # 模块化的全宽页面组件
├── snippets        # 可复用的 Liquid 代码或 HTML 片段
└── templates       # 组合 sections 以定义页面结构的模板
```

了解更多，请参阅 [主题架构文档](https://shopify.dev/docs/storefronts/themes/architecture)。

### 模板

[模板](https://shopify.dev/docs/storefronts/themes/architecture/templates#template-types)控制主题中每种类型页面上呈现的内容。

Skeleton 主题搭建了 [JSON 模板](https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates)，让商家可以轻松自定义他们的店铺。

没有任何模板类型是必需的，并且并非所有模板类型都包含在 Skeleton 主题中。完整列表请参阅 [模板类型参考](https://shopify.dev/docs/storefronts/themes/architecture/templates#template-types)。

### Sections

[Sections](https://shopify.dev/docs/storefronts/themes/architecture/sections) 是允许您创建可被商家自定义的可复用内容模块的 Liquid 文件。它们还可以包含 blocks，允许商家在 section 内添加、删除和重新排序内容。

Sections 通过在正文中包含 `{% schema %}` 来实现可定制。了解更多，请参阅 [section schema 文档](https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema)。

### Blocks

[Blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks) 让开发人员通过将 sections 分解成更小的、可复用的 Liquid 片段来创建灵活的布局。每个 block 都有自己的一组设置，并且可以在 section 内添加、删除和重新排序。

Blocks 通过在正文中包含 `{% schema %}` 来实现可定制。了解更多，请参阅 [block schema 文档](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/schema)。

## Schemas

在开发由 schema 设置定义的组件时，我们建议遵循以下指南来简化您的代码：

- **单属性设置**：对于对应单个 CSS 属性的设置，请使用 CSS 变量：

  ```liquid
  <div class="collection" style="--gap: {{ block.settings.gap }}px">
    ...
  </div>

  {% stylesheet %}
    .collection {
      gap: var(--gap);
    }
  {% endstylesheet %}

  {% schema %}
  {
    "settings": [{
      "type": "range",
      "label": "gap",
      "id": "gap",
      "min": 0,
      "max": 100,
      "unit": "px",
      "default": 0,
    }]
  }
  {% endschema %}
  ```

- **多属性设置**：对于控制多个 CSS 属性的设置，请使用 CSS 类：

  ```liquid
  <div class="collection {{ block.settings.layout }}">
    ...
  </div>

  {% stylesheet %}
    .collection--full-width {
      /* 多个样式 */
    }
    .collection--narrow {
      /* 多个样式 */
    }
  {% endstylesheet %}

  {% schema %}
  {
    "settings": [{
      "type": "select",
      "id": "layout",
      "label": "layout",
      "values": [
        { "value": "collection--full-width", "label": "t:options.full" },
        { "value": "collection--narrow", "label": "t:options.narrow" }
      ]
    }]
  }
  {% endschema %}
  ```

## CSS & JavaScript

对于 CSS 和 JavaScript，我们建议使用 [`{% stylesheet %}`](https://shopify.dev/docs/api/liquid/tags#stylesheet) 和 [`{% javascript %}`](https://shopify.dev/docs/api/liquid/tags/javascript) 标签。它们可以被包含多次，但代码只会出现一次。

### `critical.css`

Skeleton 主题明确地将每个页面必需的核心 CSS 分离到一个专用的 `critical.css` 文件中。

## 贡献

我们很高兴您为 Skeleton 主题做出贡献！此仓库旨在保持尽可能精简、轻量和基础，我们恳请您的贡献与此意图保持一致。

请访问我们的 [CONTRIBUTING.md](./CONTRIBUTING.md) 了解我们的流程、指南和建议的详细概述。

## 许可证

Skeleton 主题在 [MIT](./LICENSE.md) 许可证下开源。
