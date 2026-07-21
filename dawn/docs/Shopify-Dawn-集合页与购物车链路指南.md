# Shopify Dawn 集合页与购物车链路指南

> 接续前两篇文档，用**集合页（筛选/排序）**和**购物车（Drawer + 整页）**走通列表交互与购物流程。  
> 你会看到 `card-product` 如何在多处复用，以及加购后购物车如何无刷新更新。

**系列文档：**
- [主题架构入门指南（首页）](./Shopify-Dawn-主题架构入门指南.md)
- [产品页完整链路指南](./Shopify-Dawn-产品页完整链路指南.md)

---

# Part A：集合页（Collection）

## 和首页/产品页的区别

| 维度 | 首页 | 产品页 | **集合页** |
|---|---|---|---|
| 全局对象 | 无 | `product` | **`collection`** |
| 核心交互 | 静态展示 | 换规格 + 加购 | **筛选 + 排序 + 分页** |
| 列表组件 | featured-collection | related-products | **main-collection-product-grid** |
| 共用 snippet | card-product | card-product | **card-product + facets** |

---

## 集合页由哪 2 个 Section 组成

**文件：** `templates/collection.json`

```json
{
  "sections": {
    "banner": {
      "type": "main-collection-banner",
      "settings": {
        "show_collection_description": true,
        "show_collection_image": false
      }
    },
    "product-grid": {
      "type": "main-collection-product-grid",
      "settings": {
        "products_per_page": 16,
        "columns_desktop": 4,
        "enable_filtering": true,
        "filter_type": "horizontal",
        "enable_sorting": true
      }
    }
  },
  "order": ["banner", "product-grid"]
}
```

| Section | 职责 | 数据来源 |
|---|---|---|
| `main-collection-banner` | 合集标题、描述、头图 | **`collection.title`** / `collection.description` |
| `main-collection-product-grid` | 筛选栏 + 产品网格 + 分页 | **`collection.products`**（可被 URL 筛选参数过滤） |

---

## 完整渲染链条

```
用户访问 /collections/all（或任意合集 URL）
    ↓
Shopify 注入全局对象 collection
    ↓
layout/theme.liquid → {{ content_for_layout }}
    ↓
templates/collection.json
    ├── [1] main-collection-banner.liquid
    │       └── collection.title / collection.description（Liquid 对象）
    │
    └── [2] main-collection-product-grid.liquid
            ├── {% paginate collection.products by N %}
            ├── render 'facets'（筛选/排序 UI）
            ├── for product in collection.products
            │     └── render 'card-product'（与首页共用！）
            └── render 'pagination'
```

---

## 第 1 层 — Banner Section（简单展示）

**文件：** `sections/main-collection-banner.liquid`

```liquid
<div class="collection-hero">
  <h1 class="collection-hero__title">{{ collection.title | escape }}</h1>

  {%- if section.settings.show_collection_description -%}
    <div class="collection-hero__description rte">{{ collection.description }}</div>
  {%- endif -%}

  {%- if section.settings.show_collection_image and collection.image -%}
    <!-- 合集头图 -->
  {%- endif -%}
</div>
```

- 无 blocks，结构固定
- **`collection`** 由平台注入，不是 JSON 配置的
- 类似 Vue：`<CollectionHero :collection="collection" :show-desc="true" />`

---

## 第 2 层 — Product Grid Section（核心）

**文件：** `sections/main-collection-product-grid.liquid`

### 2.1 `paginate` 标签：分页是 Liquid 层的

```liquid
{%- paginate collection.products by section.settings.products_per_page -%}
  ...
  {%- for product in collection.products -%}
    {% render 'card-product', card_product: product, ... %}
  {%- endfor -%}

  {%- if paginate.pages > 1 -%}
    {% render 'pagination', paginate: paginate %}
  {%- endif -%}
{%- endpaginate -%}
```

**要点：**
- `paginate` 必须在 `for` 循环外层包裹
- 分页后 `collection.products` 只含**当前页**产品
- URL 带 `?page=2` 时 Shopify 自动处理

### 2.2 筛选栏：render facets snippet

```liquid
<script src="{{ 'facets.js' | asset_url }}" defer="defer"></script>

{% render 'facets',
  results: collection,
  enable_filtering: section.settings.enable_filtering,
  enable_sorting: section.settings.enable_sorting,
  filter_type: section.settings.filter_type,
  paginate: paginate
%}
```

- `results: collection` —— facets 同时支持 **collection** 和 **search** 对象
- `filter_type` 可选：`horizontal` / `vertical` / `drawer`（主题编辑器可配）

### 2.3 产品列表：再次复用 card-product

```liquid
<ul id="product-grid" data-id="{{ section.id }}">
  {%- for product in collection.products -%}
    <li class="grid__item">
      {% render 'card-product',
        card_product: product,
        section_id: section.id,
        quick_add: section.settings.quick_add,
        product_view_context: 'collection'
      %}
    </li>
  {%- endfor -%}
</ul>
```

**card-product 复用全景：**

| 出现位置 | 数据来源 |
|---|---|
| 首页 featured-collection | `section.settings.collection.products` |
| 集合页 product-grid | `collection.products`（paginate 后） |
| 产品页 related-products | `recommendations.products` |
| 搜索页 main-search | `search.results` |

改一次 `card-product.liquid`，四处一起变。

---

## 第 3 层 — facets.liquid（筛选/排序 UI）

**文件：** `snippets/facets.liquid`

```liquid
<facet-filters-form class="facets">
  <form id="FacetFiltersForm">
    {% if enable_filtering %}
      {%- for filter in results.filters -%}
        <!-- 每个 filter：价格区间 / 颜色 / 标签 / 库存等 -->
        <!-- filter.type: list / boolean / price_range -->
      {%- endfor -%}
    {% endif %}

    {% if enable_sorting %}
      <select name="sort_by">
        {%- for option in results.sort_options -%}
          <option value="{{ option.value }}">{{ option.name }}</option>
        {%- endfor -%}
      </select>
    {% endif %}
  </form>
</facet-filters-form>
```

**Liquid 对象：**
- `collection.filters` —— 当前可用的筛选维度
- `collection.sort_options` —— 排序选项
- `collection.sort_by` —— 当前 URL 中的排序值

**搜索页复用：** `sections/main-search.liquid` 同样 `render 'facets', results: search, ...`

---

## 第 4 层 — facets.js（无刷新筛选）

### 交互链路

```
用户勾选筛选 / 改排序
    ↓
<facet-filters-form> 监听 form input（debounce 800ms）
    ↓
收集 URL 查询参数（filter.v.option.color=Red&sort_by=price-ascending）
    ↓
fetch(`${pathname}?section_id=${sectionId}&${searchParams}`)
    ↓  Section Rendering API
返回 main-collection-product-grid section 的 HTML
    ↓
局部替换 3 块 DOM：
  ├── 筛选栏（renderFilters）
  ├── 产品网格（renderProductGridContainer）
  └── 产品数量（renderProductCount）
    ↓
history.pushState 更新 URL（支持浏览器后退）
```

### 关键代码（facets.js）

```javascript
static renderPage(searchParams, event, updateURLHash = true) {
  sections.forEach((section) => {
    const url = `${window.location.pathname}?section_id=${section.section}&${searchParams}`;
    fetch(url)
      .then(res => res.text())
      .then(html => FacetFiltersForm.renderSection(html, event));
  });
  if (updateURLHash) FacetFiltersForm.updateURLHash(searchParams);
}

static renderProductGridContainer(html) {
  document.getElementById('ProductGridContainer').innerHTML =
    new DOMParser().parseFromString(html, 'text/html')
      .getElementById('ProductGridContainer').innerHTML;
}
```

**Vue 等价思路：**

```javascript
// 筛选变化 → 带 query 重新 fetch 列表 HTML → 只 patch #ProductGridContainer
watch(filters, async () => {
  const html = await fetch(`/collections/shoes?section_id=product-grid&${query}`)
  patchGrid(html)
  router.replace({ query })
})
```

### 和产品页换规格的相同点

| | 产品页换规格 | 集合页筛选 |
|---|---|---|
| API | Section Rendering API | Section Rendering API |
| 请求格式 | `?section_id=xxx&option_values=...` | `?section_id=xxx&filter.v....&sort_by=...` |
| 更新方式 | 局部替换 DOM | 局部替换 DOM |
| URL 同步 | 更新 product URL | pushState 更新 collection URL |

---

## 集合页动手验证

| 步骤 | 操作 | 预期 |
|---|---|---|
| 1 | 改 `collection.json` 的 `filter_type` 为 `vertical` | 筛选栏变左侧竖排 |
| 2 | DevTools → Network，勾选颜色筛选 | 出现 `?section_id=...&filter.v...` 请求 |
| 3 | 点浏览器后退 | 列表恢复，无需整页刷新 |
| 4 | 改 `card-product.liquid` | 首页精选 + 集合页 + 搜索页一起变 |

---

# Part B：购物车（Cart Drawer + Cart Page）

## 两种购物车形态

Dawn 同时支持两种购物车 UI，由**全局设置** `settings.cart_type` 控制：

| 模式 | 触发方式 | 主要文件 |
|---|---|---|
| **Drawer（抽屉）** | 点击 header 购物车图标弹出侧栏 | `layout/theme.liquid` → `snippets/cart-drawer.liquid` |
| **Page（整页）** | 跳转 `/cart` | `templates/cart.json` → `main-cart-items.liquid` |

```
settings.cart_type == 'drawer'
    ↓
layout/theme.liquid:
  {%- render 'cart-drawer' -%}     ← 全站挂载，默认隐藏
  <script src="cart-drawer.js">

settings.cart_type == 'page'
    ↓
点击购物车图标 → 跳转 routes.cart_url（/cart）
    ↓
templates/cart.json 渲染整页购物车
```

---

## 购物车在全站架构中的位置

```
layout/theme.liquid
│
├── {% render 'cart-drawer' %}          ← Drawer 模式：全局挂载（不在 template 里）
│   └── <cart-drawer>                   ← Custom Element (cart-drawer.js)
│       └── <cart-drawer-items>         ← 行项目列表
│
├── {% sections 'header-group' %}
│   └── header.liquid
│       └── #cart-icon-bubble           ← 购物车图标 + 数量气泡
│           └── sections/cart-icon-bubble.liquid（AJAX 局部更新目标）
│
└── {{ content_for_layout }}
    └── /cart 页面时 → templates/cart.json
        ├── main-cart-items.liquid      ← <cart-items>
        └── main-cart-footer.liquid     ← 小计 + 结账按钮（blocks）
```

**关键区别：** Drawer 挂在 **Layout**，不是某个 page template；整页购物车才是 template 驱动。

---

## Drawer 模式链路

### 第 1 层 — Layout 挂载

**文件：** `layout/theme.liquid`

```liquid
{%- if settings.cart_type == 'drawer' -%}
  {%- render 'cart-drawer' -%}
{%- endif -%}

<!-- body 末尾 -->
{%- if settings.cart_type == 'drawer' -%}
  <script src="{{ 'cart-drawer.js' | asset_url }}" defer="defer"></script>
{%- endif -%}
```

### 第 2 层 — cart-drawer snippet 结构

**文件：** `snippets/cart-drawer.liquid`

```liquid
<script src="{{ 'cart.js' | asset_url }}" defer="defer"></script>

<cart-drawer class="drawer{% if cart == empty %} is-empty{% endif %}">
  <div id="CartDrawer" class="cart-drawer">
    <div id="CartDrawer-Overlay" class="cart-drawer__overlay"></div>
    <div class="drawer__inner">
      <!-- 空购物车状态 -->
      <!-- 标题 + 关闭按钮 -->
      <cart-drawer-items>
        <!-- 行项目：图片、标题、变体、数量、删除 -->
      </cart-drawer-items>
      <!-- 小计 + 结账按钮 -->
    </div>
  </div>
</cart-drawer>
```

**注意：** `sections/cart-drawer.liquid` 只有一行 `{%- render 'cart-drawer' -%}`，存在 section 是为了 **Section Rendering API** 能单独返回 drawer HTML。

### 第 3 层 — Header 购物车图标

**文件：** `sections/header.liquid`

```liquid
<a href="{{ routes.cart_url }}" id="cart-icon-bubble" class="header__icon--cart">
  <!-- cart-icon-bubble section 的内容 -->
</a>
```

**文件：** `sections/cart-icon-bubble.liquid`（极小的 section，专供 AJAX 更新）

```liquid
{% if cart == empty %}
  <span>{{ 'icon-cart-empty.svg' | inline_asset_content }}</span>
{% else %}
  <span>{{ 'icon-cart.svg' | inline_asset_content }}</span>
  <div class="cart-count-bubble">
    <span>{{ cart.item_count }}</span>
  </div>
{% endif %}
```

---

## 加购 → 更新 Drawer 完整链路

这是把**产品页**和**购物车**串起来的关键：

```
产品页点击 Add to cart
    ↓
<product-form> 拦截 submit（product-form.js）
    ↓
POST /cart/add.js
  body: FormData { id: variantId, quantity: 1, sections: ['cart-drawer', 'cart-icon-bubble'] }
    ↓
响应 JSON：
  {
    items: [...],
    sections: {
      'cart-drawer': '<div id="shopify-section-cart-drawer">...</div>',
      'cart-icon-bubble': '<div id="shopify-section-cart-icon-bubble">...</div>'
    }
  }
    ↓
publish('cart-update', { cartData: response })
    ↓
<cart-drawer>.renderContents(response)
  ├── 替换 #CartDrawer 内部 HTML
  ├── 替换 #cart-icon-bubble 数量
  └── drawer.open() 弹出抽屉
```

### getSectionsToRender 模式（Dawn 核心约定）

**文件：** `assets/cart-drawer.js`

```javascript
getSectionsToRender() {
  return [
    { id: 'cart-drawer', selector: '#CartDrawer' },
    { id: 'cart-icon-bubble' },
  ];
}

renderContents(parsedState) {
  this.getSectionsToRender().forEach((section) => {
    sectionElement.innerHTML = this.getSectionInnerHTML(
      parsedState.sections[section.id],
      section.selector
    );
  });
  this.open();  // 弹出抽屉
}
```

**设计思想：** 任何改购物车的地方，都声明「我需要刷新哪些 section」，Cart API 一次返回所有 section HTML。

| 触发来源 | 文件 | 刷新的 sections |
|---|---|---|
| 产品页加购 | product-form.js | cart-drawer + cart-icon-bubble |
| Drawer 内改数量 | cart.js (CartDrawerItems) | cart-drawer + cart-icon-bubble |
| 整页 cart 改数量 | cart.js (CartItems) | main-cart-items + cart-icon-bubble + cart-footer |
| Quick add | quick-add.js | 同上 |

---

## Pub/Sub 事件总线：跨组件同步

**文件：** `assets/pubsub.js` + `assets/constants.js`

```javascript
const PUB_SUB_EVENTS = {
  cartUpdate: 'cart-update',
  optionValueSelectionChange: 'option-value-selection-change',
  // ...
};
```

**订阅关系：**

```
product-form.js  ──publish──► cart-update ──subscribe──► cart-drawer.js
                                              ──subscribe──► cart.js (CartItems)
                                              ──subscribe──► product-info.js（刷新数量规则）
```

**举例：** Drawer 里改了数量 → publish cart-update → 产品页 quantity 显示同步更新。

---

## 整页购物车（/cart）

**文件：** `templates/cart.json`

```json
{
  "sections": {
    "cart-items": {
      "type": "main-cart-items",
      "settings": { "color_scheme": "scheme-1" }
    },
    "cart-footer": {
      "type": "main-cart-footer",
      "blocks": {
        "subtotal": { "type": "subtotal" },
        "buttons":  { "type": "buttons" }
      },
      "block_order": ["subtotal", "buttons"]
    }
  },
  "order": ["cart-items", "cart-footer"]
}
```

**文件：** `sections/main-cart-items.liquid`

```liquid
<cart-items view-event-payload="{{ cart | standard_event_data: 'view' | escape }}">
  <form action="{{ routes.cart_url }}" method="post" id="cart">
    <table>
      {%- for item in cart.items -%}
        <!-- 行项目：item.product.title, item.variant.title, item.quantity -->
      {%- endfor -%}
    </table>
  </form>
</cart-items>
```

- **`cart`** 是全局 Liquid 对象（当前购物车状态）
- `<cart-items>` → `assets/cart.js`，改数量走 `/cart/change.js`
- footer 的 blocks（subtotal / buttons）和产品页 blocks 是同一套模式

---

## Drawer vs Page 对比

| | Drawer | Page |
|---|---|---|
| 挂载位置 | `layout/theme.liquid` | `templates/cart.json` |
| 打开方式 | JS `cart-drawer.open()` | 导航到 `/cart` |
| 行项目 CE | `<cart-drawer-items>` | `<cart-items>` |
| 改数量 API | `/cart/change.js` | `/cart/change.js` |
| 刷新方式 | Section Rendering | Section Rendering |
| 空购物车 | drawer 内展示 + 可选推荐合集 | 整页空状态 |

---

## 购物车动手验证

| 步骤 | 操作 | 预期 |
|---|---|---|
| 1 | 产品页加购，看 Network | `POST /cart/add.js`，body 含 `sections=cart-drawer,cart-icon-bubble` |
| 2 | 加购后 | Drawer 弹出，header 数量更新 |
| 3 | Drawer 内改数量 | `POST /cart/change.js` |
| 4 | 主题设置改 cart_type 为 page | 加购后跳转 /cart 而非弹 drawer |
| 5 | 直接访问 `/cart?section_id=cart-drawer` | 返回 drawer section HTML（Section Rendering API） |

---

# Part C：三页串联 — 从浏览到加购

```
集合页 /collections/shoes
    │
    │  render 'card-product'（可带 quick_add）
    │
    ├─ 点击产品 → 产品页 /products/xxx
    │       │
    │       │  variant_picker + buy_buttons
    │       │
    │       └─ POST /cart/add.js
    │               ↓
    └─ quick_add（可选）─► POST /cart/add.js
                            ↓
                    cart-drawer.renderContents()
                            ↓
                    header cart-icon-bubble 更新
```

**同一条 card-product snippet，三种出口：**
1. 跳产品页（默认）
2. Quick add 原地加购（`quick_add: 'standard'`）
3. Bulk quick add 批量加购

---

## 开发者改代码决策树

```
要改什么？
│
├─ 合集标题/描述/头图
│   └─ main-collection-banner.liquid 或 collection.json settings
│
├─ 筛选/排序 UI 或行为
│   ├─ 样式/结构 → snippets/facets.liquid
│   └─ 无刷新逻辑 → assets/facets.js
│
├─ 产品卡片（全站）
│   └─ snippets/card-product.liquid
│
├─ 购物车 Drawer 结构
│   └─ snippets/cart-drawer.liquid
│
├─ 加购后行为（弹 drawer / 跳 cart）
│   ├─ config/settings_schema.json → cart_type
│   └─ assets/product-form.js
│
├─ 购物车行项目/数量/删除
│   └─ assets/cart.js
│
└─ Header 购物车数量气泡
    └─ sections/cart-icon-bubble.liquid
```

---

## 相关文件索引

### 集合页

| 文件 | 角色 |
|---|---|
| `templates/collection.json` | 页面编排 |
| `sections/main-collection-banner.liquid` | 合集头 |
| `sections/main-collection-product-grid.liquid` | 筛选 + 网格 + 分页 |
| `snippets/facets.liquid` | 筛选/排序 UI（搜索页复用） |
| `snippets/card-product.liquid` | 产品卡片 |
| `snippets/pagination.liquid` | 分页导航 |
| `assets/facets.js` | 无刷新筛选 |

### 购物车

| 文件 | 角色 |
|---|---|
| `layout/theme.liquid` | 挂载 cart-drawer |
| `snippets/cart-drawer.liquid` | Drawer HTML 结构 |
| `sections/cart-drawer.liquid` | Section Rendering 入口 |
| `sections/cart-icon-bubble.liquid` | Header 数量气泡 |
| `templates/cart.json` | 整页购物车编排 |
| `sections/main-cart-items.liquid` | 整页行项目 |
| `sections/main-cart-footer.liquid` | 小计 + 结账 |
| `assets/cart-drawer.js` | Drawer 开/关 + renderContents |
| `assets/cart.js` | 行项目数量/删除 |
| `assets/product-form.js` | 加购 → Cart API |
| `assets/pubsub.js` | 事件总线 |

---

## 延伸阅读

- [Section Rendering API](https://shopify.dev/docs/api/ajax/section-rendering)
- [Cart API](https://shopify.dev/docs/api/ajax/reference/cart)
- [Liquid collection object](https://shopify.dev/docs/api/liquid/objects/collection)
- [Liquid cart object](https://shopify.dev/docs/api/liquid/objects/cart)
- [Storefront filtering](https://shopify.dev/docs/storefronts/themes/navigation-search/filtering)

---

*文档基于 Dawn 主题项目整理。*
