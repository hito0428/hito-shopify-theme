# Shopify Dawn 主题架构入门指南

> 用 Vue 组件心智模型理解 Liquid 主题的「Layout → Template → Section → Block/Snippet → Assets」链条。  
> 案例基于本项目首页：`templates/index.json` 及相关文件。

---

## 核心结论（先看这句）

**Liquid 主题不是「一个文件 = 一个组件」，而是：平台按 JSON 配置去加载 `.liquid` 文件，再把数据灌进去。**

| Shopify (Dawn) | 近似 Vue 概念 | 关键区别 |
|---|---|---|
| `layout/theme.liquid` | `App.vue` 外壳 | 全站共用，包住所有页面 |
| `templates/index.json` | 路由 + 页面编排（放哪些组件、顺序、props 值） | **纯 JSON 配置**，不是代码 |
| `sections/image-banner.liquid` | 页面级 `.vue`（template + 局部 style + props 定义） | 一个文件里同时有 HTML、CSS、schema |
| section 内的 `blocks` | 同一 `.vue` 里的 `<slot>` / 子块 | Dawn 里 **block 不单独成文件**，写在 section 的 `{% schema %}` 里 |
| `snippets/card-product.liquid` | 可复用子组件 `<CardProduct />` | 用 `{% render %}` 显式传参，**作用域隔离** |
| `assets/*.css` / `*.js` | 独立 style / composable | section 按需 `asset_url` 引入 |

### 和 Vue 最大的不同

- **Vue**：`import Banner from './Banner.vue'` —— 代码里写引用关系
- **Shopify**：`index.json` 写 `"type": "image-banner"` —— **平台**根据 type 去找 `sections/image-banner.liquid`，并把 JSON 里的 settings 注入为 `section.settings`

---

## 目录结构速查

```
.
├── layout/          # 全站外壳（App.vue）
├── templates/       # 页面编排 JSON（路由 + 组件清单 + props）
├── sections/        # 页面级模块（.vue 组件）
├── snippets/        # 可复用碎片（子组件）
├── assets/          # CSS / JS / 图片
├── config/          # 全局主题设置
└── locales/         # 文案翻译
```

---

## 案例：访问首页 `/` 时发生了什么

### 渲染链条总览

```
用户访问 /
    ↓
Shopify 匹配 templates/index.json
    ↓
layout/theme.liquid 作为外壳渲染
    ├── {% sections 'header-group' %}     ← 固定头部（所有页面共有）
    ├── {{ content_for_layout }}          ← 插入 index.json 渲染出的 sections
    └── {% sections 'footer-group' %}     ← 固定页脚
```

---

### 第 1 层 — Layout（全站父壳）

**文件：** `layout/theme.liquid`

```liquid
{% sections 'header-group' %}

<main id="MainContent" data-template="{{ template.name }}">
  {{ content_for_layout }}
</main>

{% sections 'footer-group' %}
```

**等价 Vue：**

```vue
<template>
  <HeaderGroup />      <!-- {% sections 'header-group' %} -->
  <main>
    <router-view />    <!-- {{ content_for_layout }} ← 这里塞当前页内容 -->
  </main>
  <FooterGroup />
</template>
```

**要点：**

- `{{ content_for_layout }}` 是**占位符**，Shopify 把当前页 template 渲染出的 HTML 塞进来
- `{% sections 'header-group' %}` 引用的是 `sections/header-group.json`，不是单个 `.liquid` 文件

---

### 第 2 层 — Template JSON（页面编排，不是 HTML）

**文件：** `templates/index.json`

```json
{
  "sections": {
    "image_banner": {
      "type": "image-banner",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Browse our latest products",
            "heading_size": "h0"
          }
        },
        "button": {
          "type": "buttons",
          "settings": {
            "button_label_1": "Shop all",
            "button_link_1": "shopify://collections/all"
          }
        }
      },
      "block_order": ["heading", "button"],
      "settings": {
        "image_height": "large",
        "color_scheme": "scheme-3"
      }
    },
    "featured_collection": {
      "type": "featured-collection",
      "settings": {
        "title": "Featured products",
        "collection": "all",
        "products_to_show": 8
      }
    }
  },
  "order": ["image_banner", "featured_collection"]
}
```

**解读：**

| JSON 字段 | 运行时变成 | 作用 |
|---|---|---|
| `"type": "image-banner"` | 加载 `sections/image-banner.liquid` | 组件类型 → 文件映射 |
| `"settings"` | `section.settings.xxx` | section 级配置（props） |
| `"blocks"` + `"block_order"` | `section.blocks`（有序） | section 内子块（slot 内容） |
| `"order"` | sections 在 `<main>` 中的上下顺序 | 页面布局顺序 |

**这里没有 `import`，全是声明式配置。**

---

### 第 3 层 — Section（真正的「组件文件」）

#### 3a. `image-banner` — 带 blocks 的 section

**文件：** `sections/image-banner.liquid`

**① 引入 CSS（类似 Vue 里 `import './banner.css'`）**

```liquid
{{ 'section-image-banner.css' | asset_url | stylesheet_tag }}
```

**② 读 section 级配置**

```liquid
<div class="banner banner--{{ section.settings.image_height }}">
```

→ `section.settings.image_height` 来自 `index.json` 的 `"image_height": "large"`

**③ 循环 blocks（类似 slot 内容）**

```liquid
{%- for block in section.blocks -%}
  {%- case block.type -%}
    {%- when 'heading' -%}
      <h2 class="banner__heading {{ block.settings.heading_size }}">
        {{ block.settings.heading }}
      </h2>
    {%- when 'buttons' -%}
      <div class="banner__buttons">
        <a href="{{ block.settings.button_link_1 }}" class="button">
          {{ block.settings.button_label_1 }}
        </a>
      </div>
  {%- endcase -%}
{%- endfor -%}
```

**数据映射关系：**

```
index.json                          image-banner.liquid
─────────────────                   ─────────────────────
blocks.heading.settings.heading  →  block.settings.heading
blocks.heading.type = "heading"    →  block.type → case when 'heading'
settings.image_height = "large"    →  section.settings.image_height
```

**④ 调用 snippet（子组件）**

```liquid
{% render 'particles-banner', section_id: section.id %}
```

→ 加载 `snippets/particles-banner.liquid`  
→ 等价于 `<ParticlesBanner :section-id="section.id" />`

---

#### 3b. `featured-collection` — 带 snippet 循环的 section

**文件：** `sections/featured-collection.liquid`

```liquid
{%- for product in section.settings.collection.products limit: section.settings.products_to_show -%}
  <li class="grid__item">
    {% render 'card-product',
      card_product: product,
      media_aspect_ratio: section.settings.image_ratio,
      show_secondary_image: section.settings.show_secondary_image,
      show_vendor: section.settings.show_vendor,
      section_id: section.id,
      quick_add: section.settings.quick_add
    %}
  </li>
{%- endfor -%}
```

**数据流：**

1. `index.json` 里 `"collection": "all"` → `section.settings.collection`
2. Shopify 解析为 `collections.all` 对象
3. section 遍历 `collection.products`
4. 每个 product 交给 `card-product` snippet 渲染

---

### 第 4 层 — Snippet（纯复用碎片，无 schema）

**文件：** `snippets/card-product.liquid`

```liquid
{% comment %}
  Renders a product card
  Accepts:
  - card_product: {Object} Product Liquid object
  - show_vendor: {Boolean}
  ...
  Usage:
  {% render 'card-product', show_vendor: section.settings.show_vendor %}
{% endcomment %}
```

**特点（类似函数式组件）：**

- **没有** `{% schema %}`，商家不能在编辑器里单独配置 snippet
- 只能被 section 用 `{% render %}` 调用，参数必须**显式传入**
- 外部 `assign` 的变量**不会**自动传进来（作用域隔离）

---

### 旁支 — Section Group（Layout 里的「组件组」）

**文件：** `sections/header-group.json`

```json
{
  "sections": {
    "announcement-bar": { "type": "announcement-bar", "blocks": { ... } },
    "header": { "type": "header", "settings": { ... } }
  },
  "order": ["announcement-bar", "header"]
}
```

**引用方式（在 layout 里）：**

```liquid
{% sections 'header-group' %}
```

**等价 Vue：**

```vue
<template>
  <AnnouncementBar />
  <Header />
</template>
```

所有页面共用，不属于某个 page template。

---

## 完整链路图

```
layout/theme.liquid (App.vue)
│
├── {% sections 'header-group' %}
│   └── header-group.json
│       ├── announcement-bar.liquid
│       └── header.liquid
│
├── {{ content_for_layout }}  ← templates/index.json 渲染结果
│   │
│   ├── [1] image-banner.liquid
│   │   ├── section.settings  ← index.json 的 settings
│   │   ├── section.blocks    ← index.json 的 blocks
│   │   ├── render 'particles-banner'
│   │   └── assets/section-image-banner.css
│   │
│   └── [2] featured-collection.liquid
│       ├── section.settings.collection → collections.all.products
│       ├── render 'card-product' × N
│       └── assets/component-card.css, product-form.js ...
│
└── {% sections 'footer-group' %}
    └── footer-group.json → footer.liquid
```

---

## Vue 伪代码对照（帮助记忆）

```vue
<!-- 等价于 layout/theme.liquid + templates/index.json -->
<template>
  <HeaderGroup />

  <!-- index.json 渲染出的页面内容 -->
  <ImageBanner :image-height="'large'">
    <Heading text="Browse our latest products" />
    <Buttons label="Shop all" link="/collections/all" />
  </ImageBanner>

  <FeaturedCollection title="Featured products" collection="all">
    <CardProduct v-for="p in products" :product="p" />
  </FeaturedCollection>

  <FooterGroup />
</template>
```

**区别：** Vue 用 `import` + 标签；Shopify 用 JSON `type` + Liquid 文件 + 平台注入数据。

---

## 三个容易混淆的点

### 1. Template JSON 不包含 HTML

它只存「用哪个 section、settings 是什么」。HTML 全在 `.liquid` 里。

### 2. Dawn 的 Block 不是独立文件

`heading`、`buttons` 的类型定义在 `image-banner.liquid` 底部的 `{% schema %}`，渲染用 `case block.type`。

**不是** `blocks/heading.liquid` 那种独立文件（新版 Skeleton 主题才有 `blocks/` 目录）。

### 3. 引用关系是「平台解析」，不是 Liquid import

| 写法 | 平台行为 |
|---|---|
| JSON `"type": "image-banner"` | 找 `sections/image-banner.liquid` |
| `{% render 'card-product' %}` | 找 `snippets/card-product.liquid` |
| `{{ 'xxx.css' \| asset_url }}` | 找 `assets/xxx.css` |
| `{% sections 'header-group' %}` | 找 `sections/header-group.json` |

---

## 5 分钟动手验证

按顺序做下面 4 步，链条关系会立刻变清晰：

| 步骤 | 改什么 | 预期结果 | 验证什么 |
|---|---|---|---|
| 1 | `index.json` 里 `heading` 文案 | 首页 banner 标题变化 | JSON → `block.settings` |
| 2 | `sections/image-banner.liquid` 里 `<h2>` 改 `<h1>` | banner 标题标签变化 | section 是 HTML 模板 |
| 3 | `snippets/card-product.liquid` 价格区域 | 首页 + 集合页一起变 | snippet 跨 section 复用 |
| 4 | `layout/theme.liquid` 的 `<main>` 加边框 | 全站 main 区域变化 | layout 是最外层壳 |

---

## 各层职责一句话

| 层级 | 职责 | 谁改它 |
|---|---|---|
| **Layout** | 全站 HTML 骨架、全局 JS/CSS | 开发者 |
| **Template JSON** | 页面由哪些 section 组成、顺序、配置值 | 开发者 + 商家（主题编辑器） |
| **Section** | 页面模块的 HTML + 可配 schema + blocks | 开发者写代码，商家配 settings |
| **Block** | section 内部可拖拽子块 | 开发者定义类型，商家增删排序 |
| **Snippet** | 跨 section 复用的 UI 碎片 | 开发者 |
| **Assets** | 样式、交互脚本 | 开发者 |

---

## 下一步推荐学习路径

产品页是 Dawn 里最完整的一条链，更接近 Vue「父组件 + 多个具名 slot + 子组件 + 独立 JS」。

**详细文档：** [Shopify-Dawn-产品页完整链路指南](./Shopify-Dawn-产品页完整链路指南.md)

**继续阅读：** [Shopify-Dawn-集合页与购物车链路指南](./Shopify-Dawn-集合页与购物车链路指南.md)

```
templates/product.json
  → sections/main-product.liquid
    → blocks: title / price / variant_picker / buy_buttons ...
    → snippets/buy-buttons.liquid
    → snippets/product-variant-picker.liquid
    → snippets/price.liquid
    → assets/product-form.js
    → assets/global.js / pubsub.js
```

---

## 参考文档

- [Theme architecture](https://shopify.dev/docs/storefronts/themes/architecture)
- [JSON templates](https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates)
- [Sections](https://shopify.dev/docs/storefronts/themes/architecture/sections)
- [Snippets (render tag)](https://shopify.dev/docs/api/liquid/tags/render)
- [Liquid objects](https://shopify.dev/docs/api/liquid/objects)

---

*文档基于 Dawn 主题项目整理，案例文件：`templates/index.json`、`layout/theme.liquid`、`sections/image-banner.liquid`、`sections/featured-collection.liquid`、`snippets/card-product.liquid`。*
