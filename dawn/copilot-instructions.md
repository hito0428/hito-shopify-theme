# copilot-instructions.md

🚨 强制要求：在处理 Liquid 主题时，您必须调用一次 "learn_shopify_api"。

## 主题架构

**核心原则：专注于生成 snippets、blocks 和 sections；用户可以使用主题编辑器创建模板**

### 目录结构

```
.
├── assets          # 存储静态资源（CSS、JS、图片、字体等）
├── blocks          # 可复用、可嵌套、可自定义的组件
├── config          # 全局主题设置和自定义选项
├── layout          # 页面的顶层包装器（布局模板）
├── locales         # 主题国际化的翻译文件
├── sections        # 模块化的全宽页面组件
├── snippets        # 可复用的 Liquid 代码或 HTML 片段
└── templates       # 组合 sections 和 blocks 以定义页面结构的模板
```

#### `sections`

- Sections 是 `.liquid` 文件，允许您创建可被商家自定义的可复用模块
- Sections 可以包含 blocks，允许商家在 section 内添加、删除和重新排序内容
- Sections 通过包含必需的 `{% schema %}` 标签来实现可定制，该标签通过 JSON 对象在主题编辑器中暴露设置。使用 `schemas/section.json` JSON 模式验证该 JSON 对象
- Sections 示例：英雄横幅、产品网格、客户评价、精选合集

#### `blocks`

- Blocks 是 `.liquid` 文件，允许您创建可被商家自定义的可复用小组件（它们不需要适应页面的全宽）
- Blocks 非常适合需要复用且需要商家在主题编辑器中编辑的逻辑
- Blocks 可以包含其他嵌套的 blocks，允许商家在 block 内添加、删除和重新排序内容
- Blocks 通过包含必需的 `{% schema %}` 标签来实现可定制，该标签通过 JSON 对象在主题编辑器中暴露设置。使用 `schemas/theme_block.json` JSON 模式验证该 JSON 对象
- 如果您通过 `{% content_for 'block', id: '42', type: 'block_name' %}` 在其他文件中直接/静态地渲染 blocks，则 blocks 必须将 `{% doc %}` 标签作为头部
- Blocks 示例：单个客户评价、轮播图中的幻灯片、功能项

#### `snippets`

- Snippets 是可复用的代码片段，通过 `render` 标签在 blocks、sections 和 layout 文件中渲染
- Snippets 非常适合需要复用但商家不直接在主题编辑器中编辑的逻辑
- Snippets 在渲染时接受参数以实现动态行为
- Snippets 必须将 `{% doc %}` 标签作为头部
- Snippets 示例：按钮、元标签、CSS 变量和表单元素

#### `layout`

- 定义站点的整体 HTML 结构，包括 `<head>` 和 `<body>`，并包装其他模板以提供一致的框架
- 包含重复的全局元素，如导航、购物车抽屉、页脚，通常包括 CSS/JS 资源和元标签
- 必须包含 `{{ content_for_header }}` 以在 `<head>` 中注入 Shopify 脚本，以及 `{{ content_for_layout }}` 以渲染页面内容

#### `config`

- `config/settings_schema.json` 是一个 JSON 文件，定义全局主题设置的模式。使用 `schemas/theme_settings.json` JSON 模式验证此 JSON 文件的结构
- `config/settings_data.json` 是 JSON 文件，保存 `config/settings_schema.json` 中定义的设置的数据

#### `assets`

- 包含静态文件，如 CSS、JavaScript 和图片——包括编译和优化后的资源——通过 `asset_url` 过滤器在模板中引用
- 只在这里存放 `critical.css` 和每个页面都必需的静态文件，否则优先使用 `{% stylesheet %}` 和 `{% javascript %}` 标签

#### `locales`

- 存储按语言代码组织的翻译文件（例如，`en.default.json`、`fr.json`），以本地化所有面向用户的主题内容和编辑器字符串
- 通过在 Liquid 中提供可通过 `{{ 'key' | t }}` 等过滤器访问的翻译来启用多语言支持，以实现正确的国际化
- 使用 `schemas/translations.json` JSON 模式验证 `locales` JSON 文件

#### `templates`

- JSON 文件，定义结构、顺序以及哪些 sections 和 blocks 出现在每种页面类型上，允许商家无需修改代码即可自定义布局

### CSS & JavaScript

- 使用 `{% stylesheet %}` 和 `{% javascript %}` 标签按组件编写 CSS 和 JavaScript
- 注意：`{% stylesheet %}` 和 `{% javascript %}` 仅在 `snippets/`、`blocks/` 和 `sections/` 中受支持

### LiquidDoc

Snippets 和 blocks（当 blocks 被静态渲染时）必须包含 LiquidDoc 头部，用于记录文件的用途和所需参数。示例：

```liquid
{% doc %}
  渲染一个可能被链接包裹的响应式图片。

  @param {image} image - 要渲染的图片
  @param {string} [url] - 图片的可选目标 URL

  @example
  {% render 'image', image: product.featured_image %}
{% enddoc %}

<a href="{{ url | default: '#' }}">{{ image | image_url: width: 200, height: 200 | image_tag }}</a>
```

## blocks 和 sections 上的 `{% schema %}` 标签

**核心原则：遵循"最佳实践"并使用 JSON 模式验证 `{% schema %}` 内容**

### 最佳实践

在 sections 和 blocks 上定义 `{% schema %}` 标签时，请遵循以下指南来使用这些值：

**单属性设置**：对于对应单个 CSS 属性的设置，请使用 CSS 变量：
```liquid
<div class="collection" style="--gap: {{ block.settings.gap }}px">
  示例
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

**多属性设置**：对于控制多个 CSS 属性的设置，请使用 CSS 类：
```liquid
<div class="collection {{ block.settings.layout }}">
  示例
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

#### 移动端布局

如果您需要创建移动端布局，并且希望商家能够选择一列或两列，请使用选择输入：

```liquid
{% schema %}
{
  "type": "select",
  "id": "columns_mobile",
  "label": "移动端列数",
  "options": [
    { "value": 1, "label": "1" },
    { "value": "2", "label": "2" }
  ]
}
{% endschema %}
```

## Liquid

### Liquid 分隔符

- **`{{ ... }}`**：输出 – 打印一个值。
- **`{{- ... -}}`**：输出，修剪值周围的空白。
- **`{% ... %}`**：逻辑/控制标签（if、for、assign 等），不打印任何内容，不修剪空白。
- **`{%- ... -%}`**：逻辑/控制标签，修剪标签周围的空白。

**提示：**
在 `{%`/`{{` 之后或 `%}`/`}}` 之前添加破折号（`-`）会修剪标签旁边的空格或换行符。

**示例：**
- `{{- product.title -}}` → 打印值，移除周围的空格或换行。
- `{%- if available -%}有库存{%- endif -%}` → 逻辑，移除额外的空格/换行。

### Liquid 运算符

**比较运算符：**
- ==
- !=
- >
- <
- >=
- <=

**逻辑运算符：**
- `or`
- `and`
- `contains` - 检查字符串是否包含子字符串，或数组是否包含字符串

#### 比较和比较标签

**关键条件原则：**
- 为简单起见，当逻辑需要多个逻辑运算符时，始终使用嵌套的 `if` 条件
- Liquid 不支持括号
- Liquid 不支持三元条件，所以始终使用 `{% if cond %}`

**基本比较示例：**
```liquid
{% if product.title == "Awesome Shoes" %}
  这双鞋太棒了！
{% endif %}
```

**多个条件：**
```liquid
{% if product.type == "Shirt" or product.type == "Shoes" %}
  这是一件衬衫或一双鞋。
{% endif %}
```

**contains 用法：**
- 对于字符串：`{% if product.title contains "Pack" %}`
- 对于数组：`{% if product.tags contains "Hello" %}`
- 注意：`contains` 仅适用于字符串，不适用于数组中的对象

**{% elsif %}（仅在 if/unless 内部使用）**
```liquid
{% if a %}
  ...
{% elsif b %}
  ...
{% endif %}
```

**{% unless %}**
```liquid
{% unless condition %}
  ...
{% endunless %}
```

**{% case %}**
```liquid
{% case variable %}
  {% when 'a' %}
    a
  {% when 'b' %}
    b
  {% else %}
    其他
{% endcase %}
```

**{% else %}（在 if、unless、case 或 for 内部使用）**
```liquid
{% if product.available %}
  有库存
{% else %}
  已售罄
{% endif %}
```
_或在 for 循环内：_
```liquid
{% for item in collection.products %}
  {{ item.title }}
{% else %}
  未找到产品。
{% endfor %}
```

#### 变量和变量标签

```liquid
{% assign my_variable = 'value' %}

{% capture my_variable %}
  变量内容
{% endcapture %}

{% increment counter %}
{% decrement counter %}
```

### Liquid 过滤器

您可以在 Liquid 中链接过滤器，将一个过滤器的结果作为下一个过滤器的输入。

请参阅这些过滤器：

- `upcase`：`{{ string | upcase }}` 返回一个 **字符串**
- `split`：`{{ string | split: string }}` 返回一个 **数组**（正如我们在文档中注意到的，`split` 接收一个字符串作为其参数）
- `last`：`{{ array | last }}` 返回 **无类型**

只要类型匹配，每个过滤器都可以将其返回值传递给下一个过滤器。

例如，`upcase` 返回一个字符串，这是 `split` 的合适输入，然后 `split` 产生一个数组供 `last` 使用。

以下是过滤器如何逐步执行以最终返回 `"WORLD"` 的方式：

```liquid
{{ "hello world" | upcase | split: " " | last }}
```

- 首先，`"hello world"` 被转换为大写：`"HELLO WORLD"`，这是一个字符串
- 接下来，`split` 可以作用于字符串，所以它按空格将值拆分为数组：`["HELLO", "WORLD"]`
- 最后，`last` 过滤器处理数组，所以返回 `"WORLD"`

#### 数组
- `compact`：`{{ array | compact }}` 返回 `array`
- `concat`：`{{ array | concat: array }}` 返回 `array`
- `find`：`{{ array | find: string, string }}` 返回 `untyped`
- `find_index`：`{{ array | find_index: string, string }}` 返回 `number`
- `first`：`{{ array | first }}` 返回 `untyped`
- `has`：`{{ array | has: string, string }}` 返回 `boolean`
- `join`：`{{ array | join }}` 返回 `string`
- `last`：`{{ array | last }}` 返回 `untyped`
- `map`：`{{ array | map: string }}` 返回 `array`
- `reject`：`{{ array | reject: string, string }}` 返回 `array`
- `reverse`：`{{ array | reverse }}` 返回 `array`
- `size`：`{{ variable | size }}` 返回 `number`
- `sort`：`{{ array | sort }}` 返回 `array`
- `sort_natural`：`{{ array | sort_natural }}` 返回 `array`
- `sum`：`{{ array | sum }}` 返回 `number`
- `uniq`：`{{ array | uniq }}` 返回 `array`
- `where`：`{{ array | where: string, string }}` 返回 `array`

#### 购物车
- `item_count_for_variant`：`{{ cart | item_count_for_variant: {variant_id} }}` 返回 `number`
- `line_items_for`：`{{ cart | line_items_for: object }}` 返回 `array`

#### 合集
- `link_to_type`：`{{ string | link_to_type }}` 返回 `string`
- `link_to_vendor`：`{{ string | link_to_vendor }}` 返回 `string`
- `sort_by`：`{{ string | sort_by: string }}` 返回 `string`
- `url_for_type`：`{{ string | url_for_type }}` 返回 `string`
- `url_for_vendor`：`{{ string | url_for_vendor }}` 返回 `string`
- `within`：`{{ string | within: collection }}` 返回 `string`
- `highlight_active_tag`：`{{ string | highlight_active_tag }}` 返回 `string`

#### 颜色
- `brightness_difference`：`{{ string | brightness_difference: string }}` 返回 `number`
- `color_brightness`：`{{ string | color_brightness }}` 返回 `number`
- `color_contrast`：`{{ string | color_contrast: string }}` 返回 `number`
- `color_darken`：`{{ string | color_darken: number }}` 返回 `string`
- `color_desaturate`：`{{ string | color_desaturate: number }}` 返回 `string`
- `color_difference`：`{{ string | color_difference: string }}` 返回 `number`
- `color_extract`：`{{ string | color_extract: string }}` 返回 `number`
- `color_lighten`：`{{ string | color_lighten: number }}` 返回 `string`
- `color_mix`：`{{ string | color_mix: string, number }}` 返回 `string`
- `color_modify`：`{{ string | color_modify: string, number }}` 返回 `string`
- `color_saturate`：`{{ string | color_saturate: number }}` 返回 `string`
- `color_to_hex`：`{{ string | color_to_hex }}` 返回 `string`
- `color_to_hsl`：`{{ string | color_to_hsl }}` 返回 `string`
- `color_to_oklch`：`{{ string | color_to_oklch }}` 返回 `string`
- `color_to_rgb`：`{{ string | color_to_rgb }}` 返回 `string`
- `hex_to_rgba`：`{{ string | hex_to_rgba }}` 返回 `string`

#### 客户
- `customer_login_link`：`{{ string | customer_login_link }}` 返回 `string`
- `customer_logout_link`：`{{ string | customer_logout_link }}` 返回 `string`
- `customer_register_link`：`{{ string | customer_register_link }}` 返回 `string`
- `avatar`：`{{ customer | avatar }}` 返回 `string`
- `login_button`：`{{ shop | login_button }}` 返回 `string`

#### 日期
- `date`：`{{ date | date: string }}` 返回 `string`

#### 默认
- `default_errors`：`{{ string | default_errors }}` 返回 `string`
- `default`：`{{ variable | default: variable }}` 返回 `untyped`
- `default_pagination`：`{{ paginate | default_pagination }}` 返回 `string`

#### 字体
- `font_face`：`{{ font | font_face }}` 返回 `string`
- `font_modify`：`{{ font | font_modify: string, string }}` 返回 `font`
- `font_url`：`{{ font | font_url }}` 返回 `string`

#### 格式
- `date`：`{{ string | date: string }}` 返回 `string`
- `json`：`{{ variable | json }}` 返回 `string`
- `structured_data`：`{{ variable | structured_data }}` 返回 `string`
- `unit_price_with_measurement`：`{{ number | unit_price_with_measurement: unit_price_measurement }}` 返回 `string`
- `weight_with_unit`：`{{ number | weight_with_unit }}` 返回 `string`

#### 托管文件
- `asset_img_url`：`{{ string | asset_img_url }}` 返回 `string`
- `asset_url`：`{{ string | asset_url }}` 返回 `string`
- `file_img_url`：`{{ string | file_img_url }}` 返回 `string`
- `file_url`：`{{ string | file_url }}` 返回 `string`
- `global_asset_url`：`{{ string | global_asset_url }}` 返回 `string`
- `shopify_asset_url`：`{{ string | shopify_asset_url }}` 返回 `string`

#### Html
- `time_tag`：`{{ string | time_tag: string }}` 返回 `string`
- `inline_asset_content`：`{{ asset_name | inline_asset_content }}` 返回 `string`
- `highlight`：`{{ string | highlight: string }}` 返回 `string`
- `link_to`：`{{ string | link_to: string }}` 返回 `string`
- `placeholder_svg_tag`：`{{ string | placeholder_svg_tag }}` 返回 `string`
- `preload_tag`：`{{ string | preload_tag: as: string }}` 返回 `string`
- `script_tag`：`{{ string | script_tag }}` 返回 `string`
- `stylesheet_tag`：`{{ string | stylesheet_tag }}` 返回 `string`

#### 本地化
- `currency_selector`：`{{ form | currency_selector }}` 返回 `string`
- `translate`：`{{ string | t }}` 返回 `string`
- `format_address`：`{{ address | format_address }}` 返回 `string`

#### 数学
- `abs`：`{{ number | abs }}` 返回 `number`
- `at_least`：`{{ number | at_least }}` 返回 `number`
- `at_most`：`{{ number | at_most }}` 返回 `number`
- `ceil`：`{{ number | ceil }}` 返回 `number`
- `divided_by`：`{{ number | divided_by: number }}` 返回 `number`
- `floor`：`{{ number | floor }}` 返回 `number`
- `minus`：`{{ number | minus: number }}` 返回 `number`
- `modulo`：`{{ number | modulo: number }}` 返回 `number`
- `plus`：`{{ number | plus: number }}` 返回 `number`
- `round`：`{{ number | round }}` 返回 `number`
- `times`：`{{ number | times: number }}` 返回 `number`

#### 媒体
- `external_video_tag`：`{{ variable | external_video_tag }}` 返回 `string`
- `external_video_url`：`{{ media | external_video_url: attribute: string }}` 返回 `string`
- `image_tag`：`{{ string | image_tag }}` 返回 `string`
- `media_tag`：`{{ media | media_tag }}` 返回 `string`
- `model_viewer_tag`：`{{ media | model_viewer_tag }}` 返回 `string`
- `video_tag`：`{{ media | video_tag }}` 返回 `string`
- `article_img_url`：`{{ variable | article_img_url }}` 返回 `string`
- `collection_img_url`：`{{ variable | collection_img_url }}` 返回 `string`
- `image_url`：`{{ variable | image_url: width: number, height: number }}` 返回 `string`
- `img_tag`：`{{ string | img_tag }}` 返回 `string`
- `img_url`：`{{ variable | img_url }}` 返回 `string`
- `product_img_url`：`{{ variable | product_img_url }}` 返回 `string`

#### 元字段
- `metafield_tag`：`{{ metafield | metafield_tag }}` 返回 `string`
- `metafield_text`：`{{ metafield | metafield_text }}` 返回 `string`

#### 货币
- `money`：`{{ number | money }}` 返回 `string`
- `money_with_currency`：`{{ number | money_with_currency }}` 返回 `string`
- `money_without_currency`：`{{ number | money_without_currency }}` 返回 `string`
- `money_without_trailing_zeros`：`{{ number | money_without_trailing_zeros }}` 返回 `string`

#### 支付
- `payment_button`：`{{ form | payment_button }}` 返回 `string`
- `payment_terms`：`{{ form | payment_terms }}` 返回 `string`
- `payment_type_img_url`：`{{ string | payment_type_img_url }}` 返回 `string`
- `payment_type_svg_tag`：`{{ string | payment_type_svg_tag }}` 返回 `string`

#### 字符串
- `blake3`：`{{ string | blake3 }}` 返回 `string`
- `hmac_sha1`：`{{ string | hmac_sha1: string }}` 返回 `string`
- `hmac_sha256`：`{{ string | hmac_sha256: string }}` 返回 `string`
- `md5`：`{{ string | md5 }}` 返回 `string`
- `sha1`：`{{ string | sha1: string }}` 返回 `string`
- `sha256`：`{{ string | sha256: string }}` 返回 `string`
- `append`：`{{ string | append: string }}` 返回 `string`
- `base64_decode`：`{{ string | base64_decode }}` 返回 `string`
- `base64_encode`：`{{ string | base64_encode }}` 返回 `string`
- `base64_url_safe_decode`：`{{ string | base64_url_safe_decode }}` 返回 `string`
- `base64_url_safe_encode`：`{{ string | base64_url_safe_encode }}` 返回 `string`
- `capitalize`：`{{ string | capitalize }}` 返回 `string`
- `downcase`：`{{ string | downcase }}` 返回 `string`
- `escape`：`{{ string | escape }}` 返回 `string`
- `escape_once`：`{{ string | escape_once }}` 返回 `string`
- `lstrip`：`{{ string | lstrip }}` 返回 `string`
- `newline_to_br`：`{{ string | newline_to_br }}` 返回 `string`
- `prepend`：`{{ string | prepend: string }}` 返回 `string`
- `remove`：`{{ string | remove: string }}` 返回 `string`
- `remove_first`：`{{ string | remove_first: string }}` 返回 `string`
- `remove_last`：`{{ string | remove_last: string }}` 返回 `string`
- `replace`：`{{ string | replace: string, string }}` 返回 `string`
- `replace_first`：`{{ string | replace_first: string, string }}` 返回 `string`
- `replace_last`：`{{ string | replace_last: string, string }}` 返回 `string`
- `rstrip`：`{{ string | rstrip }}` 返回 `string`
- `slice`：`{{ string | slice }}` 返回 `string`
- `split`：`{{ string | split: string }}` 返回 `array`
- `strip`：`{{ string | strip }}` 返回 `string`
- `strip_html`：`{{ string | strip_html }}` 返回 `string`
- `strip_newlines`：`{{ string | strip_newlines }}` 返回 `string`
- `truncate`：`{{ string | truncate: number }}` 返回 `string`
- `truncatewords`：`{{ string | truncatewords: number }}` 返回 `string`
- `upcase`：`{{ string | upcase }}` 返回 `string`
- `url_decode`：`{{ string | url_decode }}` 返回 `string`
- `url_encode`：`{{ string | url_encode }}` 返回 `string`
- `camelize`：`{{ string | camelize }}` 返回 `string`
- `handleize`：`{{ string | handleize }}` 返回 `string`
- `url_escape`：`{{ string | url_escape }}` 返回 `string`
- `url_param_escape`：`{{ string | url_param_escape }}` 返回 `string`
- `pluralize`：`{{ number | pluralize: string, string }}` 返回 `string`

#### 标签
- `link_to_add_tag`：`{{ string | link_to_add_tag }}` 返回 `string`
- `link_to_remove_tag`：`{{ string | link_to_remove_tag }}` 返回 `string`
- `link_to_tag`：`{{ string | link_to_tag }}` 返回 `string`

### Liquid 对象

#### 全局对象
- `collections`
- `pages`
- `all_products`
- `articles`
- `blogs`
- `cart`
- `closest`
- `content_for_header`
- `customer`
- `images`
- `linklists`
- `localization`
- `metaobjects`
- `request`
- `routes`
- `shop`
- `theme`
- `settings`
- `template`
- `additional_checkout_buttons`
- `all_country_option_tags`
- `canonical_url`
- `content_for_additional_checkout_buttons`
- `content_for_index`
- `content_for_layout`
- `country_option_tags`
- `current_page`
- `handle`
- `page_description`
- `page_image`
- `page_title`
- `powered_by_link`
- `scripts`

#### `/article` 页面
- `article`
- `blog`

#### `/blog` 页面
- `blog`
- `current_tags`

#### `/cart` 页面
- `cart`

#### `/checkout` 页面
- `checkout`

#### `/collection` 页面
- `collection`
- `current_tags`

#### `/customers/account` 页面
- `customer`

#### `/customers/addresses` 页面
- `customer`

#### `/customers/order` 页面
- `customer`
- `order`

#### `/gift_card.liquid` 页面
- `gift_card`
- `recipient`

#### `/metaobject` 页面
- `metaobject`

#### `/page` 页面
- `page`

#### `/product` 页面
- `product`
- `remote_product`

#### `/robots.txt.liquid` 页面
- `robots`

#### `/search` 页面
- `search`
### Liquid 标签


#### content_for
`content_for` 标签需要一个 type 参数来区分渲染多个主题 blocks（`'blocks'`）和单个静态 block（`'block'`）。


语法：
```
{% content_for 'blocks' %}
{% content_for 'block', type: "slide", id: "slide-1" %}
```

#### form
因为 Shopify 主题中有许多不同的表单类型可用，所以 `form` 标签需要一个类型。根据表单类型，可能需要额外的参数。您可以指定以下表单类型：

- [`activate_customer_password`](https://shopify.dev/docs/api/liquid/tags/form#form-activate_customer_password)
- [`cart`](https://shopify.dev/docs/api/liquid/tags/form#form-cart)
- [`contact`](https://shopify.dev/docs/api/liquid/tags/form#form-contact)
- [`create_customer`](https://shopify.dev/docs/api/liquid/tags/form#form-create_customer)
- [`currency`](https://shopify.dev/docs/api/liquid/tags/form#form-currency)
- [`customer`](https://shopify.dev/docs/api/liquid/tags/form#form-customer)
- [`customer_address`](https://shopify.dev/docs/api/liquid/tags/form#form-customer_address)
- [`customer_login`](https://shopify.dev/docs/api/liquid/tags/form#form-customer_login)
- [`guest_login`](https://shopify.dev/docs/api/liquid/tags/form#form-guest_login)
- [`localization`](https://shopify.dev/docs/api/liquid/tags/form#form-localization)
- [`new_comment`](https://shopify.dev/docs/api/liquid/tags/form#form-new_comment)
- [`product`](https://shopify.dev/docs/api/liquid/tags/form#form-product)
- [`recover_customer_password`](https://shopify.dev/docs/api/liquid/tags/form#form-recover_customer_password)
- [`reset_customer_password`](https://shopify.dev/docs/api/liquid/tags/form#form-reset_customer_password)
- [`storefront_password`](https://shopify.dev/docs/api/liquid/tags/form#form-storefront_password)


语法：
```
{% form 'form_type' %}
  内容
{% endform %}
```

#### layout

语法：
```
{% layout name %}
```

#### assign
您可以创建任何 [基本类型](https://shopify.dev/docs/api/liquid/basics#types)、[对象](https://shopify.dev/docs/api/liquid/objects) 或对象属性的变量。

> 注意：
> 预定义的 Liquid 对象可能会被同名变量覆盖。
> 为确保您可以访问所有 Liquid 对象，请确保您的变量名与预定义对象的名称不匹配。


语法：
```
{% assign variable_name = value %}
```

#### break

语法：
```
{% break %}
```

#### capture
您可以使用 Liquid 逻辑和变量创建复杂的字符串。

> 注意：
> 预定义的 Liquid 对象可能会被同名变量覆盖。
> 为确保您可以访问所有 Liquid 对象，请确保您的变量名与预定义对象的名称不匹配。


语法：
```
{% capture variable %}
  值
{% endcapture %}
```

#### case

语法：
```
{% case variable %}
  {% when first_value %}
    first_expression
  {% when second_value %}
    second_expression
  {% else %}
    third_expression
{% endcase %}
```

#### comment
`comment` 标签内的任何文本都不会被输出，任何 Liquid 代码都会被解析但不会被执行。


语法：
```
{% comment %}
  内容
{% endcomment %}
```

#### continue

语法：
```
{% continue %}
```

#### cycle
`cycle` 标签必须在 `for` 循环内使用。

> 提示：
> 使用 `cycle` 标签以可预测的模式输出文本。例如，为表格中的行应用奇/偶类。


语法：
```
{% cycle string, string, ... %}
```

#### decrement
使用 `decrement` 声明的变量对于创建它们的 [layout](/themes/architecture/layouts)、[template](/themes/architecture/templates) 或 [section](/themes/architecture/sections) 文件是唯一的。但是，该变量在文件中包含的 [snippets](/themes/architecture/snippets) 之间共享。

同样，使用 `decrement` 创建的变量与使用 [`assign`](https://shopify.dev/docs/api/liquid/tags/assign) 和 [`capture`](https://shopify.dev/docs/api/liquid/tags/capture) 创建的变量是独立的。但是，`decrement` 和 [`increment`](https://shopify.dev/docs/api/liquid/tags/increment) 共享变量。


语法：
```
{% decrement variable_name %}
```

#### doc
`doc` 标签允许开发人员在 Liquid 模板中包含文档。`doc` 标签内的任何内容都不会被渲染或输出。其中的 Liquid 代码将被解析但不会被执行。这有助于为代码补全、代码检查和内联文档等功能提供工具支持。

有关详细的文档语法和示例，请参阅 [`LiquidDoc` 参考](https://shopify.dev/docs/storefronts/themes/tools/liquid-doc)。


语法：
```
{% doc %}
  渲染一条消息。

  @param {string} foo - 一个字符串值。
  @param {string} [bar] - 一个可选的字符串值。

  @example
  {% render 'message', foo: 'Hello', bar: 'World' %}
{% enddoc %}
```

#### echo
使用 `echo` 标签与将表达式包装在花括号（`{{` 和 `}}`）中相同。但是，与花括号方法不同，您可以在 [`liquid` 标签](https://shopify.dev/docs/api/liquid/tags/liquid) 内使用 `echo` 标签。

> 提示：
> 您可以在 `echo` 标签内对表达式使用 [过滤器](https://shopify.dev/docs/api/liquid/filters)。


语法：
```
{% liquid
  echo expression
%}
```

#### for
使用 `for` 循环最多可以进行 50 次迭代。如果您需要迭代超过 50 个项目，则使用 [`paginate` 标签](https://shopify.dev/docs/api/liquid/tags/paginate) 将项目拆分到多个页面。

> 提示：
> 每个 `for` 循环都有一个关联的 [`forloop` 对象](https://shopify.dev/docs/api/liquid/objects/forloop)，其中包含有关循环的信息。


语法：
```
{% for variable in array %}
  表达式
{% endfor %}
```

#### if

语法：
```
{% if condition %}
  表达式
{% endif %}
```

#### increment
使用 `increment` 声明的变量对于创建它们的 [layout](/themes/architecture/layouts)、[template](/themes/architecture/templates) 或 [section](/themes/architecture/sections) 文件是唯一的。但是，该变量在文件中包含的 [snippets](/themes/architecture/snippets) 之间共享。

同样，使用 `increment` 创建的变量与使用 [`assign`](https://shopify.dev/docs/api/liquid/tags/assign) 和 [`capture`](https://shopify.dev/docs/api/liquid/tags/capture) 创建的变量是独立的。但是，`increment` 和 [`decrement`](https://shopify.dev/docs/api/liquid/tags/decrement) 共享变量。


语法：
```
{% increment variable_name %}
```

#### raw

语法：
```
{% raw %}
  表达式
{% endraw %}
```

#### render
在 snippets 和 app blocks 内部，您不能直接访问在 snippet 或 app block 外部 [创建](https://shopify.dev/docs/api/liquid/tags/variable-tags) 的变量。但是，您可以 [将变量指定为参数](https://shopify.dev/docs/api/liquid/tags/render#render-passing-variables-to-a-snippet)，以将外部变量传递给 snippets。

虽然您不能直接访问创建的变量，但您可以访问全局对象，以及在 snippet 或 app block 外部可直接访问的任何对象。例如，[产品模板](/themes/architecture/templates/product) 内的 snippet 或 app block 可以访问 [`product` 对象](https://shopify.dev/docs/api/liquid/objects/product)，而 [section](/themes/architecture/sections) 内的 snippet 或 app block 可以访问 [`section` 对象](https://shopify.dev/docs/api/liquid/objects/section)。

在 snippet 或 app block 外部，您不能访问在 snippet 或 app block 内部创建的变量。

> 注意：
> 当您使用 `render` 标签渲染 snippet 时，不能在 snippet 内部使用 [`include` 标签](https://shopify.dev/docs/api/liquid/tags/include)。


语法：
```
{% render 'filename' %}
```

#### tablerow
`tablerow` 标签必须包裹在 HTML `<table>` 和 `</table>` 标签中。

> 提示：
> 每个 `tablerow` 循环都有一个关联的 [`tablerowloop` 对象](https://shopify.dev/docs/api/liquid/objects/tablerowloop)，其中包含有关循环的信息。


语法：
```
{% tablerow variable in array %}
  表达式
{% endtablerow %}
```

#### unless
> 提示：
> 与 [`if` 标签](https://shopify.dev/docs/api/liquid/tags/if) 类似，您可以使用 `elsif` 为 `unless` 标签添加更多条件。


语法：
```
{% unless condition %}
  表达式
{% endunless %}
```

#### paginate
因为 [`for` 循环](https://shopify.dev/docs/api/liquid/tags/for) 每页限制为 50 次迭代，所以您需要使用 `paginate` 标签来迭代包含超过 50 个项目的数组。以下数组可以分页：

- [`article.comments`](https://shopify.dev/docs/api/liquid/objects/article#article-comments)
- [`blog.articles`](https://shopify.dev/docs/api/liquid/objects/blog#blog-articles)
- [`collections`](https://shopify.dev/docs/api/liquid/objects/collections)
- [`collection.products`](https://shopify.dev/docs/api/liquid/objects/collection#collection-products)
- [`customer.addresses`](https://shopify.dev/docs/api/liquid/objects/customer#customer-addresses)
- [`customer.orders`](https://shopify.dev/docs/api/liquid/objects/customer#customer-orders)
- [`metaobject_definition.values`](https://shopify.dev/docs/api/liquid/objects/metaobject_definition#metaobject_definition-values)
- [`pages`](https://shopify.dev/docs/api/liquid/objects/pages)
- [`product.variants`](https://shopify.dev/docs/api/liquid/objects/product#variants)
- [`search.results`](https://shopify.dev/docs/api/liquid/objects/search#search-results)
- [`article_list` 设置](/themes/architecture/settings/input-settings#article_list)
- [`collection_list` 设置](/themes/architecture/settings/input-settings#collection_list)
- [`product_list` 设置](/themes/architecture/settings/input-settings#product_list)

在 `paginate` 标签内，您可以访问 [`paginate` 对象](https://shopify.dev/docs/api/liquid/objects/paginate)。您可以使用此对象或 [`default_pagination` 过滤器](https://shopify.dev/docs/api/liquid/filters/default_pagination) 来构建页面导航。

> 注意：
> `paginate` 标签允许用户分页到数组中的第 25,000 个项目，不能更远。要访问数组中更远的项目，应在分页之前进一步过滤数组。有关更多信息，请参阅 [分页限制](/themes/best-practices/performance/platform#pagination-limits)。


语法：
```
{% paginate array by page_size %}
  {% for item in array %}
    forloop_content
  {% endfor %}
{% endpaginate %}
```

#### javascript
每个 section、block 或 snippet 只能有一个 `{% javascript %}` 标签。

要了解有关在 `javascript` 标签之间定义的 JavaScript 如何加载和运行的更多信息，请参阅 [javascript 标签](/storefronts/themes/best-practices/javascript-and-stylesheet-tags#javascript) 文档。
> 注意：
> Liquid 不会在 `{% javascript %}` 标签内渲染。包含 Liquid 代码可能会导致语法错误。


语法：
```
{% javascript %}
  javascript_code
{% endjavascript %}
```

#### section
使用 `section` 标签渲染 section 会静态地渲染一个 section。要了解有关 sections 以及如何在主题中使用它们的更多信息，请参阅 [渲染 section](/themes/architecture/sections#render-a-section)。


语法：
```
{% section 'name' %}
```

#### stylesheet
每个 section、block 或 snippet 只能有一个 `{% stylesheet %}` 标签。

要了解有关在 `stylesheet` 标签之间定义的 CSS 如何加载和运行的更多信息，请参阅 [stylesheet 标签](/storefronts/themes/best-practices/javascript-and-stylesheet-tags#stylesheet) 文档。
> 注意：
> Liquid 不会在 `{% stylesheet %}` 标签内渲染。包含 Liquid 代码可能会导致语法错误。


语法：
```
{% stylesheet %}
  css_styles
{% endstylesheet %}
```

#### sections
使用此标签将 section 组渲染为主题 [layout](/themes/architecture/layouts) 内容的一部分。将 `sections` 标签放在您希望在布局中渲染它的位置。

要了解有关 section 组以及如何在主题中使用它们的更多信息，请参阅 [Section 组](/themes/architecture/section-groups#usage)。


语法：
```
{% sections 'name' %}
```

#### style
> 注意：
> 如果您在 `style` 标签内引用 [颜色设置](/themes/architecture/settings/input-settings#color)，则关联的 CSS 规则将随着主题编辑器中设置的更改而更新，无需刷新页面。


语法：
```
{% style %}
  CSS_rules
{% endstyle %}
```

#### else
您可以将 `else` 标签与以下标签一起使用：

- [`case`](https://shopify.dev/docs/api/liquid/tags/case)
- [`if`](https://shopify.dev/docs/api/liquid/tags/if)
- [`unless`](https://shopify.dev/docs/api/liquid/tags/unless)


语法：
```
{% else %}
  表达式
```

#### else

语法：
```
{% for variable in array %}
  first_expression
{% else %}
  second_expression
{% endfor %}
```

#### liquid
因为标签没有分隔符，所以每个标签需要单独占一行。

> 提示：
> 使用 [`echo` 标签](https://shopify.dev/docs/api/liquid/tags/echo) 在 `liquid` 标签内输出表达式。


语法：
```
{% liquid
  表达式
%}
```


## 翻译开发标准

### 翻译要求

- **每个面向用户的文本**都必须使用翻译过滤器。
- **更新 `locales/en.default.json`** 以包含所有新键。
- **使用描述性的分层键**进行组织。
- **只添加英文文本**；翻译人员处理其他语言。

### 翻译过滤器用法

**对所有文本使用 `{{ 'key' | t }}`：**

```liquid
<!-- 好 -->
<h2>{{ 'sections.featured_collection.title' | t }}</h2>
<p>{{ 'sections.featured_collection.description' | t }}</p>
<button>{{ 'products.add_to_cart' | t }}</button>

<!-- 不好 -->
<h2>精选合集</h2>
<p>查看我们最好的产品</p>
<button>加入购物车</button>
```

### 带变量的翻译

**使用变量进行插值：**

```liquid
<!-- Liquid 模板 -->
<p>{{ 'products.price_range' | t: min: product.price_min | money, max: product.price_max | money }}</p>
<p>{{ 'general.pagination.page' | t: page: paginate.current_page, pages: paginate.pages }}</p>
```

**区域设置文件中的对应键：**

```json
{
  "products": {
    "price_range": "从 {{ min }} 到 {{ max }}"
  },
  "general": {
    "pagination": {
      "page": "第 {{ page }} 页，共 {{ pages }} 页"
    }
  }
}
```

### 最佳实践

**内容指南：**
- 书写清晰、简洁的文本。
- 对所有面向用户的文本使用**句首大写**，包括标题、标题和按钮标签（仅将第一个单词和专有名词大写；例如，`Featured collection` → `精选合集`，而不是 `精选合集`）。
- 保持术语一致。
- 考虑 UI 元素的字符限制。

**变量用法：**
- 使用插值而不是将字符串拼接在一起。
- 变量命名优先考虑清晰度而不是简洁性。
- 转义变量，除非它们输出 HTML：`{{ variable | escape }}`。


## 本地化标准

在 `locales/` 目录中工作时自动附加。

### 文件结构

```
locales/
├── en.default.json          # 英文（必需）
├── en.default.schema.json   # 英文（必需）
├── es.json                  # 西班牙语
├── est.schema.json          # 西班牙语
├── fr.json                  # 法语
├── frt.schema.json          # 法语
└── pt-BR.json               # 葡萄牙语
└── pt-BR..schema.json       # 葡萄牙语
```

#### 区域设置文件

区域设置文件是 JSON 文件，包含整个 Shopify 主题及其编辑器中使用的所有文本字符串的翻译。它们让商家可以轻松更新和本地化重复的单词和短语，使得将店铺内容和设置翻译成多种语言以供国际客户使用成为可能。这些文件提供了一种集中管理和编辑翻译的方式。

**示例：**
```json
{
  "general": {
    "cart": "购物车",
    "checkout": "结账"
  },
  "products": {
    "add_to_cart": "加入购物车"
  }
}
```

#### Schema 区域设置文件

Schema 区域设置文件以 .schema.json 扩展名保存，专门存储主题编辑器设置 schema 的翻译字符串。它们遵循结构化的组织——类别、组和描述——为每个翻译提供上下文，从而实现编辑器内容的准确定位。Schema 区域设置文件在命名时必须使用 IETF 语言标签格式，例如 en-GB.schema.json 表示英式英语或 fr-CA.schema.json 表示加拿大法语。

**示例：**
```json
{
  "products": {
    "card": {
      "description": "产品卡片布局"
    }
  }
}
```

### 键组织

**分层结构：**
```json
{
  "general": {
    "meta": {
      "title": "{{ shop_name }}",
      "description": "{{ shop_description }}"
    },
    "accessibility": {
      "skip_to_content": "跳转到内容",
      "close": "关闭"
    }
  },
  "products": {
    "add_to_cart": "加入购物车",
    "quick_view": "快速查看",
    "price": {
      "regular": "常规价格",
      "sale": "促销价",
      "unit": "单价"
    }
  }
}
```
**用法**
```liquid
{{ 'general.meta.title' | t: shop_name: shop.name }}
{{ 'general.meta.description' | t: shop_description: shop.description }}
```

### 翻译指南

**键命名：**
- 使用描述性的分层键
- 最多 3 层深度
- 键名使用 snake_case
- 对相关翻译进行分组

**内容规则：**
- 保持 UI 元素的文本简洁
- 对动态内容使用变量
- 考虑字符限制
- 保持一致的术语

## 每种资产类型的示例

### `snippet`

```liquid
{% doc %}
  渲染一个可能被链接包裹的响应式图片。

  当提供 `width`、`height` 和 `crop` 时，图片将以固定的宽高比渲染。

  作为如何使用 `image_url` 过滤器和 `image_tag` 过滤器以及如何使用 LiquidDoc 记录代码的示例。

  @param {image} image - 要渲染的图片
  @param {string} [url] - 图片的可选目标 URL
  @param {string} [css_class] - 要添加到图片包装器的可选类
  @param {number} [width] - 要渲染的图片的最高分辨率宽度
  @param {number} [height] - 要渲染的图片的最高分辨率高度
  @param {string} [crop] - 图片的裁剪位置

  @example
  {% render 'image', image: product.featured_image %}
  {% render 'image', image: product.featured_image, url: product.url %}
  {% render 'image',
    css_class: 'product__image',
    image: product.featured_image,
    url: product.url,
    width: 1200,
    height: 800,
    crop: 'center',
  %}
{% enddoc %}

{% liquid
  unless height
    assign width = width | default: image.width
  endunless

  if url
    assign wrapper = 'a'
  else
    assign wrapper = 'div'
  endif
%}

<{{ wrapper }}
  class="image {{ css_class }}"
  {% if url %}
    href="{{ url }}"
  {% endif %}
>
  {{ image | image_url: width: width, height: height, crop: crop | image_tag }}
</{{ wrapper }}>

{% stylesheet %}
  .image {
    display: block;
    position: relative;
    overflow: hidden;
    width: 100%;
    height: auto;
  }

  .image > img {
    width: 100%;
    height: auto;
  }
{% endstylesheet %}

{% javascript %}
  function doSomething() {
    // 示例
  }
  doSomething()
{% endjavascript %}

```

### `block`

#### 文本

```liquid
{% doc %}
  渲染一个文本 block。

  @example
  {% content_for 'block', type: 'text', id: 'text' %}
{% enddoc %}

<div
  class="text {{ block.settings.text_style }}"
  style="--text-align: {{ block.settings.alignment }}"
  {{ block.shopify_attributes }}
>
  {{ block.settings.text }}
</div>

{% stylesheet %}
  .text {
    text-align: var(--text-align);
  }
  .text--title {
    font-size: 2rem;
    font-weight: 700;
  }
  .text--subtitle {
    font-size: 1.5rem;
  }
{% endstylesheet %}

{% schema %}
{
  "name": "t:general.text",
  "settings": [
    {
      "type": "text",
      "id": "text",
      "label": "t:labels.text",
      "default": "文本"
    },
    {
      "type": "select",
      "id": "text_style",
      "label": "t:labels.text_style",
      "options": [
        { "value": "text--title", "label": "t:options.text_style.title" },
        { "value": "text--subtitle", "label": "t:options.text_style.subtitle" },
        { "value": "text--normal", "label": "t:options.text_style.normal" }
      ],
      "default": "text--title"
    },
    {
      "type": "text_alignment",
      "id": "alignment",
      "label": "t:labels.alignment",
      "default": "left"
    }
  ],
  "presets": [{ "name": "t:general.text" }]
}
{% endschema %}
```

#### 组

```liquid
{% doc %}
  渲染一组 blocks，具有可配置的布局方向、间距和对齐方式。

  所有设置仅应用于一个维度，以降低配置复杂性。

  此组件是一个包装器，仅关注以指定的布局方向渲染其子元素，并具有适当的内边距和对齐方式。

  @example
  {% content_for 'block', type: 'group', id: 'group' %}
{% enddoc %}

<div
  class="group {{ block.settings.layout_direction }}"
  style="
    --padding: {{ block.settings.padding }}px;
    --alignment: {{ block.settings.alignment }};
  "
  {{ block.shopify_attributes }}
>
  {% content_for 'blocks' %}
</div>

{% stylesheet %}
  .group {
    display: flex;
    flex-wrap: nowrap;
    overflow: hidden;
    width: 100%;
  }
  .group--horizontal {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    padding: 0 var(--padding);
  }
  .group--vertical {
    flex-direction: column;
    align-items: var(--alignment);
    padding: var(--padding) 0;
  }
{% endstylesheet %}

{% schema %}
{
  "name": "t:general.group",
  "blocks": [{ "type": "@theme" }],
  "settings": [
    {
      "type": "select",
      "id": "layout_direction",
      "label": "t:labels.layout_direction",
      "default": "group--vertical",
      "options": [
        { "value": "group--horizontal", "label": "t:options.direction.horizontal" },
        { "value": "group--vertical", "label": "t:options.direction.vertical" }
      ]
    },
    {
      "visible_if": "{{ block.settings.layout_direction == 'group--vertical' }}",
      "type": "select",
      "id": "alignment",
      "label": "t:labels.alignment",
      "default": "flex-start",
      "options": [
        { "value": "flex-start", "label": "t:options.alignment.left" },
        { "value": "center", "label": "t:options.alignment.center" },
        { "value": "flex-end", "label": "t:options.alignment.right" }
      ]
    },
    {
      "type": "range",
      "id": "padding",
      "label": "t:labels.padding",
      "default": 0,
      "min": 0,
      "max": 200,
      "step": 2,
      "unit": "px"
    }
  ],
  "presets": [
    {
      "name": "t:general.column",
      "category": "t:general.layout",
      "settings": {
        "layout_direction": "group--vertical",
        "alignment": "flex-start",
        "padding": 0
      }
    },
    {
      "name": "t:general.row",
      "category": "t:general.layout",
      "settings": {
        "layout_direction": "group--horizontal",
        "padding": 0
      }
    }
  ]
}
{% endschema %}
```

### `section`

```liquid
<div class="example-section full-width">
  {% if section.settings.background_image %}
    <div class="example-section__background">
      {{ section.settings.background_image | image_url: width: 2000 | image_tag }}
    </div>
  {% endif %}

  <div class="custom-section__content">
    {% content_for 'blocks' %}
  </div>
</div>

{% stylesheet %}
  .example-section {
    position: relative;
    overflow: hidden;
    width: 100%;
  }
  .example-section__background {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: -1;
    overflow: hidden;
  }
  .example-section__background img {
    position: absolute;
    width: 100%;
    height: auto;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  .example-section__content {
    display: grid;
    grid-template-columns: var(--content-grid);
  }

  .example-section__content > * {
    grid-column: 2;
  }
{% endstylesheet %}

{% schema %}
{
  "name": "t:general.custom_section",
  "blocks": [{ "type": "@theme" }],
  "settings": [
    {
      "type": "image_picker",
      "id": "background_image",
      "label": "t:labels.background"
    }
  ],
  "presets": [
    {
      "name": "t:general.custom_section"
    }
  ]
}
{% endschema %}
```
