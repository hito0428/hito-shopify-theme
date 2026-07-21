# Shopify Dawn 产品页完整链路指南

> 接续《Shopify-Dawn-主题架构入门指南》，用产品页走通「Template → Section → Blocks → Snippets → JS 交互」全链路。  
> 案例基于：`templates/product.json` 及关联文件。

---

## 和首页案例的区别（先看这个）

首页主要是 **静态展示 + snippet 复用**；产品页多了三层：

| 新增能力 | 涉及文件 | 作用 |
|---|---|---|
| **Liquid 全局对象 `product`** | 平台注入，非 JSON 配置 | 标题、variants、media、description 等 |
| **Blocks 是可拖拽的功能模块** | `main-product.liquid` + `product.json` | 商家可增删排序：价格、规格、加购按钮… |
| **JS 无刷新更新** | `product-info.js` + `product-form.js` | 换规格更新价格；加购更新购物车 |

产品页更接近 Vue 的完整形态：**父组件 + 多个具名 slot + 子组件 + composable + 事件总线**。

---

## 产品页由哪 3 个 Section 组成

**文件：** `templates/product.json`

```json
{
  "sections": {
    "main":              { "type": "main-product",     "blocks": { ... }, "block_order": [...] },
    "disclosures":       { "type": "disclosures",      "settings": { ... } },
    "related-products":  { "type": "related-products", "settings": { ... } }
  },
  "order": ["main", "disclosures", "related-products"]
}
```

| Section | 职责 | Vue 类比 |
|---|---|---|
| `main-product` | 左图右信息、规格选择、加购 | 页面主组件 `<ProductDetail />` |
| `disclosures` | 合规/披露信息 | 独立功能块 |
| `related-products` | 「You may also like」推荐 | 列表子组件，复用 `card-product` |

---

## 完整渲染链条

```
用户访问 /products/xxx
    ↓
Shopify 注入全局对象 product（当前商品数据）
    ↓
layout/theme.liquid
    ├── header-group（同上篇）
    ├── {{ content_for_layout }}
    │   └── templates/product.json 按 order 渲染 3 个 section
    │       ├── [1] main-product.liquid      ← 核心
    │       ├── [2] disclosures.liquid
    │       └── [3] related-products.liquid  ← 复用 card-product snippet
    └── footer-group
```

---

## 第 1 层 — Template JSON：Blocks 就是「具名 Slot」

**文件：** `templates/product.json`（`main` section 部分）

```json
{
  "type": "main-product",
  "blocks": {
    "vendor":           { "type": "text",             "settings": { "text": "{{ product.vendor }}" } },
    "title":            { "type": "title",            "settings": {} },
    "price":            { "type": "price",            "settings": {} },
    "variant_picker":   { "type": "variant_picker",   "settings": { "picker_type": "button" } },
    "quantity_selector":{ "type": "quantity_selector","settings": {} },
    "buy_buttons":      { "type": "buy_buttons",      "settings": { "show_dynamic_checkout": true } },
    "description":      { "type": "description",      "settings": {} },
    "share":            { "type": "share",            "settings": { "share_label": "Share" } }
  },
  "block_order": [
    "vendor", "title", "price", "variant_picker",
    "quantity_selector", "buy_buttons", "description", "share"
  ],
  "settings": {
    "media_size": "large",
    "gallery_layout": "stacked",
    "media_position": "left",
    "enable_sticky_info": true
  }
}
```

**Vue 等价写法：**

```vue
<ProductDetail
  :media-size="'large'"
  :gallery-layout="'stacked'"
  :sticky="true"
>
  <Vendor />           <!-- block: text -->
  <Title />            <!-- block: title -->
  <Price />            <!-- block: price -->
  <VariantPicker />    <!-- block: variant_picker -->
  <QuantitySelector /> <!-- block: quantity_selector -->
  <BuyButtons />       <!-- block: buy_buttons -->
  <Description />      <!-- block: description -->
  <Share />            <!-- block: share -->
</ProductDetail>
```

**和首页 banner blocks 的相同点：** JSON 里的 `blocks` → 运行时 `section.blocks`，按 `block_order` 排序。  
**不同点：** 产品页 blocks 类型更多（10+ 种），且大量逻辑拆到 snippet 和 JS。

---

## 第 2 层 — main-product.liquid：Section 主体结构

**文件：** `sections/main-product.liquid`

### 2.1 外层 Web Component 壳

```liquid
<product-info
  id="MainProduct-{{ section.id }}"
  data-section="{{ section.id }}"
  data-product-id="{{ product.id }}"
  data-update-url="true"
  data-url="{{ product.url }}"
>
  ...
</product-info>
```

- `<product-info>` 是 Custom Element，对应 `assets/product-info.js`
- `data-update-url="true"`：换规格时更新浏览器 URL
- **`product` 对象**由 Shopify 平台注入（不是 JSON 配的），包含 `product.title`、`product.variants`、`product.media` 等

### 2.2 左右两栏布局

```liquid
<div class="product grid grid--2-col-tablet">
  <!-- 左：媒体画廊（不在 blocks 里，section 固定结构） -->
  <div class="product__media-wrapper">
    {% render 'product-media-gallery', variant_images: variant_images %}
  </div>

  <!-- 右：信息区（blocks 循环渲染） -->
  <div class="product__info-wrapper">
    {%- for block in section.blocks -%}
      {%- case block.type -%}
        ...
      {%- endcase -%}
    {%- endfor -%}
  </div>
</div>
```

**要点：**

- **媒体画廊**是 section 固定结构，不在 `block_order` 里
- **右侧信息区**完全由 blocks 驱动，商家可在主题编辑器里拖拽排序

### 2.3 引入 Assets（CSS + JS）

```liquid
{{ 'section-main-product.css' | asset_url | stylesheet_tag }}
{{ 'component-price.css' | asset_url | stylesheet_tag }}

<script src="{{ 'product-info.js' | asset_url }}" defer="defer"></script>
<script src="{{ 'product-form.js' | asset_url }}" defer="defer"></script>
```

类似 Vue 里：

```vue
<script setup>
import './section-main-product.css'
import { useProductInfo } from './product-info.js'
import { useProductForm } from './product-form.js'
</script>
```

---

## 第 3 层 — Blocks 分发：`case block.type`

**文件：** `sections/main-product.liquid`（核心 switch）

| block.type | 渲染方式 | 数据来源 |
|---|---|---|
| `text` | section 内联 HTML | `block.settings.text` |
| `title` | section 内联 HTML | **`product.title`**（Liquid 对象） |
| `price` | `{% render 'price' %}` | **`product`** + 当前 variant |
| `variant_picker` | `{% render 'product-variant-picker' %}` | **`product.options_with_values`** |
| `quantity_selector` | section 内联 + `<quantity-input>` | variant 数量规则 |
| `buy_buttons` | `{% render 'buy-buttons' %}` | form + JS 加购 |
| `description` | section 内联 HTML | **`product.description`** |
| `share` | `{% render 'share-button' %}` | block settings |
| `@app` | `{% render block %}` | App Block 扩展点 |

### 典型 block → snippet 调用

```liquid
{%- when 'price' -%}
  <div id="price-{{ section.id }}">
    {%- render 'price',
      product: product,
      use_variant: true,
      show_badges: true,
      price_class: 'price--large'
    -%}
  </div>

{%- when 'variant_picker' -%}
  {% render 'product-variant-picker',
    product: product,
    block: block,
    product_form_id: product_form_id
  %}

{%- when 'buy_buttons' -%}
  {%- render 'buy-buttons',
    block: block,
    product: product,
    product_form_id: product_form_id,
    section_id: section.id,
    show_pickup_availability: true
  -%}
```

**关键变量 `product_form_id`：**

```liquid
{%- assign product_form_id = 'product-form-' | append: section.id -%}
```

把 variant picker、quantity input、buy form **绑到同一个 form**（通过 HTML `form="product-form-xxx"` 属性）。

---

## 第 4 层 — 关键 Snippets 拆解

### 4.1 `price.liquid` — 纯展示组件

```liquid
{%- liquid
  if use_variant
    assign target = product.selected_or_first_available_variant
  else
    assign target = product
  endif
  assign price = target.price
  assign compare_at_price = target.compare_at_price
-%}

<div class="price price--large {% if compare_at_price > price %}price--on-sale{% endif %}">
  ...
  {{ money_price }}
</div>
```

- 无 schema，纯函数式
- `use_variant: true` → 显示**当前选中规格**的价格（换规格后会被 JS 局部替换）

---

### 4.2 `product-variant-picker.liquid` — 规格选择器

```liquid
<variant-selects
  id="variant-selects-{{ section.id }}"
  data-section="{{ section.id }}"
  data-product-id="{{ product.id }}"
>
  {%- for option in product.options_with_values -%}
    <fieldset class="product-form__input">
      <legend>{{ option.name }}</legend>
      {% render 'product-variant-options',
        product: product,
        option: option,
        block: block,
        picker_type: block.settings.picker_type
      %}
    </fieldset>
  {%- endfor -%}
</variant-selects>
```

- `<variant-selects>` → `assets/global.js` 里的 Custom Element
- 每个 option（颜色/尺码）再调 `product-variant-options` snippet
- input 带 `form="{{ product_form_id }}"`，和加购表单联动

---

### 4.3 `buy-buttons.liquid` — 加购表单

```liquid
<product-form class="product-form" data-section-id="{{ section.id }}">
  {%- form 'product', product, id: product_form_id, data-type: 'add-to-cart-form' -%}
    <input type="hidden" name="id" value="{{ product.selected_or_first_available_variant.id }}" class="product-variant-id">

    <div class="product-form__buttons">
      <button type="submit" name="add">Add to cart</button>
      {%- if show_dynamic_checkout -%}
        {{ form | payment_button }}
      {%- endif -%}
    </div>
  {%- endform -%}
</product-form>
```

- `{% form 'product' %}` → Shopify 标准产品表单
- 隐藏字段 `name="id"` 存 **variant id**（换规格时 JS 更新这个值）
- `<product-form>` → `assets/product-form.js`

---

## 第 5 层 — JS 交互：两条核心链路

Dawn 用 **Custom Elements + Pub/Sub 事件总线**，不是 Vue 的 reactive。

### 全局事件总线

**文件：** `assets/pubsub.js` + `assets/constants.js`

```javascript
const PUB_SUB_EVENTS = {
  cartUpdate: 'cart-update',
  optionValueSelectionChange: 'option-value-selection-change',
  cartError: 'cart-error',
};

function subscribe(eventName, callback) { ... }
function publish(eventName, data) { ... }
```

类似 Vue 的 mitt / eventBus。

---

### 链路 A：换规格 → 无刷新更新价格/库存/按钮

```
用户点击颜色/尺码
    ↓
<variant-selects> change 事件（global.js）
    ↓
publish('option-value-selection-change', { selectedOptionValues })
    ↓
<product-info> 订阅该事件（product-info.js）
    ↓
fetch(`${product.url}?section_id=${sectionId}&option_values=...`)
    ↓  Section Rendering API：只返回该 section 的 HTML
parse HTML → 局部替换 price / variant-selects / buy-buttons 等区域
    ↓
更新 URL、隐藏 input 的 variant id、刷新按钮 disabled 状态
```

**关键代码（product-info.js）：**

```javascript
handleOptionValueChange({ data: { event, target, selectedOptionValues } }) {
  this.renderProductInfo({
    requestUrl: this.buildRequestUrlWithParams(productUrl, selectedOptionValues),
    callback: this.handleUpdateProductInfo(productUrl),
  });
}

renderProductInfo({ requestUrl, callback }) {
  fetch(requestUrl)
    .then(res => res.text())
    .then(html => callback(new DOMParser().parseFromString(html, 'text/html')));
}
```

**Vue 等价思路：**

```javascript
// 换规格 → 重新 fetch 局部 HTML → patch DOM
watch(selectedVariant, async () => {
  const html = await fetch(`/products/xxx?section_id=main`)
  patchPrice(html)
  patchInventory(html)
})
```

---

### 链路 B：点击 Add to cart → 更新购物车抽屉

```
用户点击 Submit
    ↓
<product-form> 拦截 submit（product-form.js）
    ↓
fetch(routes.cart_add_url, { body: FormData(form) })   ← Cart AJAX API
    ↓
publish('cart-update', { cartData: response })
    ↓
<cart-drawer> 订阅 cart-update → renderContents(response)
    ↓
购物车抽屉弹出，数量气泡更新
```

**关键代码（product-form.js）：**

```javascript
onSubmitHandler(evt) {
  evt.preventDefault();
  const formData = new FormData(this.form);
  formData.append('sections', this.cart.getSectionsToRender().map(s => s.id));

  fetch(`${routes.cart_add_url}`, { body: formData })
    .then(res => res.json())
    .then(response => {
      publish(PUB_SUB_EVENTS.cartUpdate, {
        source: 'product-form',
        productVariantId: formData.get('id'),
        cartData: response,
      });
      this.cart.renderContents(response);
    });
}
```

**和换规格链路的区别：**

| | 换规格 | 加购 |
|---|---|---|
| API | Section Rendering API | Cart Add API |
| 返回 | HTML 片段 | JSON（含 cart + sections HTML） |
| 更新目标 | 产品页局部 DOM | 购物车 drawer + icon bubble |

---

## 第 6 层 — related-products：跨 Section 复用 Snippet

**文件：** `sections/related-products.liquid`

```liquid
<product-recommendations
  data-url="{{ routes.product_recommendations_url }}?limit=4"
  data-product-id="{{ product.id }}"
>
  {% for recommendation in recommendations.products %}
    {% render 'card-product',
      card_product: recommendation,
      show_secondary_image: section.settings.show_secondary_image,
      product_view_context: 'recommendation'
    %}
  {% endfor %}
</product-recommendations>
```

和首页 `featured-collection` **复用同一个** `snippets/card-product.liquid`，只是数据来源不同：

| Section | 产品数据来源 |
|---|---|
| `featured-collection` | `section.settings.collection.products`（商家选的合集） |
| `related-products` | Shopify 推荐 API（`recommendations.products`） |
| `main-product` complementary block | 互补推荐 API |

---

## 全链路一图

```
layout/theme.liquid
│
└── templates/product.json
    │
    ├── main-product.liquid  (<product-info>)
    │   │
    │   ├── [固定] render 'product-media-gallery'
    │   │
    │   └── [blocks 循环]
    │       ├── text        → 内联（block.settings）
    │       ├── title       → 内联（product.title）
    │       ├── price       → snippet price.liquid
    │       ├── variant_picker → snippet product-variant-picker.liquid
    │       │                    └── snippet product-variant-options.liquid
    │       │                    └── <variant-selects> → global.js
    │       ├── quantity_selector → <quantity-input> + form 属性关联
    │       ├── buy_buttons → snippet buy-buttons.liquid
    │       │                    └── <product-form> → product-form.js
    │       │                    └── {% form 'product' %}
    │       ├── description → 内联（product.description）
    │       └── share       → snippet share-button.liquid
    │
    │   JS 交互层
    │   ├── product-info.js  ← 订阅 optionValueSelectionChange
    │   ├── product-form.js  ← 拦截 submit → cart_add_url
    │   ├── pubsub.js        ← 事件总线
    │   └── global.js        ← variant-selects / quantity-input 等
    │
    ├── disclosures.liquid
    │
    └── related-products.liquid
        └── render 'card-product' × N  （与首页共用 snippet）
```

---

## Form 联动：多个 Block 如何组成一个表单

产品页最「反直觉」的设计：**blocks 在 DOM 上是平级的，但逻辑上属于同一个 form**。

```
┌─ product-form-123 ─────────────────────────────┐
│  variant_picker 的 radio/select  form="product-form-123"  │
│  quantity 的 input                 form="product-form-123"  │
│  buy_buttons 的 {% form 'product' id: product-form-123 %} │
│  hidden input name="id"  ← variant id，JS 换规格时更新     │
└──────────────────────────────────────────────┘
```

HTML5 的 `form="xxx"` 属性让**表单控件可以放在 form 标签外面**，Dawn 靠这个实现 blocks 自由排序。

---

## 5 分钟动手验证（产品页版）

| 步骤 | 改什么 | 预期 | 验证 |
|---|---|---|---|
| 1 | `product.json` 里调换 `price` 和 `title` 的 block_order | 价格跑到标题上面 | blocks 顺序 = DOM 顺序 |
| 2 | `snippets/price.liquid` 加一行测试文案 | 产品页价格区变化 | snippet 被 main-product 调用 |
| 3 | 浏览器 DevTools 换规格，看 Network | 出现 `?section_id=...&option_values=...` 请求 | Section Rendering API |
| 4 | 点 Add to cart，看 Network | 出现 `/cart/add.js` 请求 | Cart AJAX API |
| 5 | `related-products.liquid` 改标题 | 仅底部推荐区变化 | 同一 template 下 section 独立 |

---

## 首页 vs 产品页架构对比

| 维度 | 首页 (index.json) | 产品页 (product.json) |
|---|---|---|
| 全局 Liquid 对象 | 无特殊对象 | **`product`** |
| Section 数量 | 2 个 | 3 个 |
| Blocks 复杂度 | 2 种（heading/buttons） | 10+ 种 |
| Snippet 深度 | 1 层（card-product） | 2~3 层（variant-options 等） |
| JS 交互 | 较少 | **换规格 + 加购 + 推荐加载** |
| Form 联动 | 无 | **跨 block 的 form 属性** |
| API 调用 | 无 | Section Rendering + Cart AJAX |

---

## 开发者改产品页时的决策树

```
要改什么？
│
├─ 商家可配的布局/文案/开关
│   └─ 改 section 的 {% schema %} 或 block schema
│
├─ 某个功能块的 HTML 结构
│   ├─ 简单块 → 改 main-product.liquid 里对应的 when 分支
│   └─ 复杂/复用 → 改或新建 snippet
│
├─ 换规格 / 加购 / 购物车行为
│   └─ 改 assets/product-info.js 或 product-form.js
│
├─ 产品卡片样式（多处共用）
│   └─ 改 snippets/card-product.liquid
│
└─ 页面有哪些 section、blocks 顺序
    └─ 改 templates/product.json（或主题编辑器）
```

---

## 相关文件索引

| 文件 | 角色 |
|---|---|
| `templates/product.json` | 页面编排 + blocks 配置 |
| `sections/main-product.liquid` | 产品页主 section |
| `snippets/price.liquid` | 价格展示 |
| `snippets/product-variant-picker.liquid` | 规格选择器 |
| `snippets/product-variant-options.liquid` | 单个 option 的 radio/select |
| `snippets/buy-buttons.liquid` | 加购表单 |
| `snippets/product-media-gallery.liquid` | 媒体画廊 |
| `snippets/card-product.liquid` | 产品卡片（related-products 复用） |
| `assets/product-info.js` | 换规格 → 局部刷新 |
| `assets/product-form.js` | 加购 → Cart API |
| `assets/global.js` | variant-selects、quantity-input 等 CE |
| `assets/pubsub.js` | 事件总线 |
| `assets/constants.js` | 事件名常量 |

---

## 延伸阅读

- [上一篇：Shopify-Dawn-主题架构入门指南](./Shopify-Dawn-主题架构入门指南.md)
- [下一篇：Shopify-Dawn-集合页与购物车链路指南](./Shopify-Dawn-集合页与购物车链路指南.md)
- [Section Rendering API](https://shopify.dev/docs/api/ajax/section-rendering)
- [Cart API - Add line item](https://shopify.dev/docs/api/ajax/reference/cart#post-locale-cart-add-js)
- [Liquid product object](https://shopify.dev/docs/api/liquid/objects/product)
- [form 标签 (product)](https://shopify.dev/docs/api/liquid/tags/form#form-product)

---

*文档基于 Dawn 主题项目整理。*
