---
name: email-design-system
description: Build and modify EmailShepherd Email Design System (EDS) components, fields, templates, and design tokens. Use when working on EDS repositories — defining components, writing Liquid templates, configuring field definitions, or setting up design tokens and custom styles.
metadata:
  author: emailshepherd
  version: "1.0.0"
---

# EmailShepherd Email Design System — Agent Reference

This is a reference for AI coding agents working on EmailShepherd Email Design System (EDS) repositories.

An EDS defines the drag-and-drop email editor experience. The components you define become the blocks users drag into emails. The fields you define become the editable controls in the editor sidebar. Labels, groups, hints, and visibility rules all directly shape the editor UI.

For exact TypeScript types, look up `node_modules/@emailshepherd/eds-sdk/dist/types.d.ts` in the project.

## Defining a Component

A component is defined using `defineComponent` and consists of a name, label, description, field definitions, and a Liquid template.

The `description` is important — EmailShepherd's AI Agents use it to decide which components to select. Write it to explain what the component is for and when it should be used (e.g. "Full-width image with headline and CTA. Use as the first component in promotional emails").

```typescript
import { defineComponent } from '@emailshepherd/eds-sdk/types';
import template from './template.liquid?raw';

export default defineComponent({
  name: "component_name",       // Unique identifier
  label: "Component Label",     // Display name in the editor
  description: "What this component does and when to use it",
  field_definitions: [
    // ... fields (see Field Types below)
  ],
  feed_id: null,
  deprecated: false,
  template: template
});
```

The template is HTML with [Liquid](https://shopify.dev/docs/storefronts/themes/liquid/reference) syntax. Fields are referenced using their `liquid_variable`:

```html
<table>
  <tr>
    <td>
      <p>{{headline}}</p>
      {% if show_cta %}
        <a href="{{cta_url}}">{{cta_text}}</a>
      {% endif %}
    </td>
  </tr>
</table>
```

## Field Types

All fields share these common properties:

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | yes | Field type (see types below) |
| `label` | string | yes | Display label in the editor |
| `liquid_variable` | string | yes | Variable name used in templates |
| `default_value` | varies | yes | Initial value |
| `group` | string | no | Groups related fields visually in the editor |
| `visible_if` | string | no | Liquid expression controlling visibility |
| `hint` | string | no | Tooltip text in the editor |
| `validations` | object | no | Type-specific validation rules |
| `feed_field_name` | string | no | Maps to a data feed column |

### `text`

Simple text input. Default value is a string.

```typescript
{
  type: "text",
  label: "Headline",
  liquid_variable: "headline",
  default_value: "Welcome aboard"
}
```

Validations: `min_length`, `max_length`, `must_not_be_blank`, `must_not_be_default`

### `number`

Number input. Default value is a number.

```typescript
{
  type: "number",
  label: "Image width",
  liquid_variable: "img_width",
  default_value: 600
}
```

Validations: `min`, `max`

### `boolean`

Toggle. Default value is `true` or `false`. Commonly used to show/hide optional sections.

```typescript
{
  type: "boolean",
  label: "Show CTA",
  liquid_variable: "show_cta",
  default_value: true
}
```

### `color`

Color picker. Default value is a hex color string.

```typescript
{
  type: "color",
  label: "Background color",
  liquid_variable: "bg_color",
  default_value: "#FFFFFF"
}
```

### `enum`

Dropdown with predefined options. Default value must match one of the option values.

```typescript
{
  type: "enum",
  label: "Text alignment",
  liquid_variable: "text_alignment",
  default_value: "center",
  options: [
    { label: "Left", value: "left" },
    { label: "Center", value: "center" },
    { label: "Right", value: "right" }
  ]
}
```

### `image`

Image uploader / URL input. Default value is a URL string.

```typescript
{
  type: "image",
  label: "Hero image",
  liquid_variable: "hero_image",
  default_value: "https://placehold.co/600x300"
}
```

Validations: `must_not_be_blank`, `must_not_be_default`

### `url`

URL input. Has an optional `skip_link_tracking` property.

```typescript
{
  type: "url",
  label: "CTA link",
  liquid_variable: "cta_url",
  default_value: "https://example.com"
}
```

Validations: `must_not_be_blank`, `must_not_be_default`

### `rich_text`

Rich text editor. The `marks` property controls which formatting options are available. If `marks` is `null`, all formatting is disabled.

```typescript
{
  type: "rich_text",
  label: "Body content",
  liquid_variable: "body_content",
  default_value: "Default text",
  marks: {
    bold: { enabled: true },
    italic: { enabled: true },
    link: { enabled: true },
    bullet_list: { enabled: true },
    numbered_list: { enabled: true }
  }
}
```

Validations: `min_content_length`, `max_content_length`, `must_not_be_blank`, `must_not_be_default`

To allow custom styles on a rich text field, add the `custom_styles_names` array referencing style names defined in `custom_styles.ts`:

```typescript
{
  type: "rich_text",
  // ...
  custom_styles_names: ["brand_blue", "price_highlight"]
}
```

### `horizontal_align`

Horizontal alignment picker. Options must be a subset of `left`, `center`, `right`. Default value must match one of the provided options.

```typescript
{
  type: "horizontal_align",
  label: "Text alignment",
  liquid_variable: "text_alignment",
  default_value: "center",
  options: ["left", "center", "right"]
}
```

### `vertical_align`

Vertical alignment picker. Options must be a subset of `top`, `middle`, `bottom`. Default value must match one of the provided options.

```typescript
{
  type: "vertical_align",
  label: "Content alignment",
  liquid_variable: "content_alignment",
  default_value: "middle",
  options: ["top", "middle", "bottom"]
}
```

### `code`

Raw code / HTML editor. Default value is a string.

```typescript
{
  type: "code",
  label: "Custom HTML",
  liquid_variable: "custom_html",
  default_value: "<p>Hello</p>"
}
```

Validations: `must_not_be_blank`, `must_not_be_default`

## Field Groups

Use the `group` property to group related fields together in the editor:

```typescript
{
  type: "image",
  label: "Source",
  liquid_variable: "image_src",
  default_value: "https://placehold.co/600x300",
  group: "Feature Image"
},
{
  type: "text",
  label: "Alt text",
  liquid_variable: "image_alt",
  default_value: "Feature image",
  group: "Feature Image"
}
```

## Field Visibility

Use `visible_if` to conditionally show/hide fields. The value is a Liquid expression (without `{% %}` tags):

```typescript
{
  type: "boolean",
  label: "Show CTA",
  liquid_variable: "show_cta",
  default_value: true
},
{
  type: "text",
  label: "CTA text",
  liquid_variable: "cta_text",
  default_value: "Learn more",
  visible_if: "show_cta == true"
},
{
  type: "url",
  label: "CTA URL",
  liquid_variable: "cta_url",
  default_value: "https://example.com",
  visible_if: "show_cta == true"
}
```

## Templates

Templates support full [Liquid](https://shopify.dev/docs/storefronts/themes/liquid/reference) syntax — conditionals, loops, filters, etc.

### Referencing Fields

| Scope | Syntax |
| --- | --- |
| Local field | `{{field_name}}` |
| Container (global) field | `{{global_fields.field_name}}` |
| Design token | `{{render_context.design_tokens.colors.primary}}` |
| Email subject | `{{render_context.email.content.subject}}` |
| Email preheader | `{{render_context.email.content.preheader}}` |

### Editor Focus

Add `data-es-field-focus` to template elements so that clicking them in the editor focuses the corresponding field:

```html
<h1 data-es-field-focus="headline">{{headline}}</h1>
<div data-es-field-focus="body_text">{{body_text}}</div>
```

## Container Component

The container component wraps all emails. Its template **must** include the `{{children}}` placeholder — this is where user-added components are inserted.

Container fields are **global fields**. Any component's template can reference container fields using the `global_fields` prefix — for example `{{global_fields.logo_url}}`.

Minimal container template:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{render_context.email.content.subject}}</title>
  </head>
  <body>
    <div style="display:none">{{render_context.email.content.preheader}}</div>
    <table role="presentation" width="100%">
      <tr>
        <td align="center">
          <table role="presentation" width="600" style="max-width:600px">
            <tr>
              <td>
                {{children}}
              </td>
            </tr>
          </table>
        </td>
      </tr>
    </table>
  </body>
</html>
```

## Design Tokens

Design tokens are a freeform JSON object for reusable design values (colors, spacing, typography, etc.) that can be referenced in templates.

```typescript
import { defineDesignTokens } from '@emailshepherd/eds-sdk/types';

export default defineDesignTokens({
  colors: {
    primary: "#0066CC",
    secondary: "#FF6B35",
    text: "#212529",
    background: "#FFFFFF"
  },
  spacing: {
    small: "8px",
    medium: "16px",
    large: "24px"
  },
  typography: {
    font_family: "Arial, Helvetica, sans-serif",
    heading_size: "24px",
    body_size: "16px"
  }
});
```

Reference in templates: `{{render_context.design_tokens.colors.primary}}`

Use design tokens for values that should be consistent across components but are not user-editable fields.

## Custom Styles

Custom styles define named text styles available in `rich_text` fields. They can be used in two ways.

```typescript
import { defineCustomStyles } from '@emailshepherd/eds-sdk/types';

export default defineCustomStyles([
  {
    name: "brand_highlight",     // Referenced by name in field definitions
    label: "Brand Highlight",    // Display label in the editor
    style: "color: #0066CC; font-weight: bold;"  // Inline CSS
  }
]);
```

### As a custom mark

Add the style's `name` to a rich text field's `custom_styles_names` array. It will appear as a selectable style option in the email editor's rich text toolbar. When applied to text, it renders as a `<span>` with the style's CSS:

```html
<span class="es-custom-style es-custom-style-brand_highlight" data-custom-style="brand_highlight" style="color: #0066CC; font-weight: bold;">styled text</span>
```

### As a built-in mark override

Each built-in mark (`bold`, `italic`, `link`, `bullet_list`, `numbered_list`) accepts an optional `custom_style_name` that references a custom style. The custom style's CSS is then applied as inline styles on the built-in mark's tag:

```typescript
{
  type: "rich_text",
  label: "Body",
  liquid_variable: "body",
  default_value: "Text",
  marks: {
    bold: { enabled: true, custom_style_name: "brand_highlight" },
    italic: { enabled: true },
    link: { enabled: true },
    bullet_list: { enabled: false },
    numbered_list: { enabled: false }
  }
}
```

This would render bold text as:

```html
<strong class="es-bold" style="color: #0066CC; font-weight: bold;">bold text</strong>
```

### Rich text HTML output

Only the following tags are permitted in rich text content:

| Element | Tag | CSS class |
| --- | --- | --- |
| Bold | `<strong>` | `es-bold` |
| Italic | `<em>` | `es-italic` |
| Bullet list | `<ul>` | `es-bullet-list` |
| Ordered list | `<ol>` | `es-ordered-list` |
| List item | `<li>` | `es-list-item` |
| Link | `<a>` | `es-link` |
| Custom style | `<span>` | `es-custom-style es-custom-style-{name}` |

Custom style `<span>` tags may only reference styles that are listed in the field's `custom_styles_names`. Arbitrary `<span>` tags are not permitted.

## Render Context

Every render provides a `render_context` object with these properties:

```liquid
render_context.render_mode                       # "export" | "render_api" | "email_design_system_preview"
render_context.output_format                     # "html" | "plaintext"
render_context.email.id
render_context.email.name
render_context.email.locale
render_context.email.content.subject
render_context.email.content.preheader
render_context.project.id
render_context.project.name
render_context.email_design_system.id
render_context.email_design_system.name
render_context.brand_profile.id           # nil if no brand profile
render_context.brand_profile.name
render_context.design_tokens.*            # merged design tokens
render_context.component_instance.id
render_context.component_instance.name
render_context.component_instance.component_name
render_context.previous_component_instance.*   # nil if first
render_context.next_component_instance.*       # nil if last
render_context.connector.id
render_context.connector.name
render_context.connector.type
```

`render_mode` can be used to hide content in the editor preview that should only appear in the export output. This is useful when you need to embed ESP code or other content that would look ugly in the preview:

```html
{% if render_context.render_mode == 'export' %}
  {% raw %}{{ first_name }}{% endraw %}
{% else %}
  John
{% endif %}
```

`output_format` can be used to provide different content for HTML vs plain-text exports:

```html
{% if render_context.output_format == "plaintext" %}
  <div>Plain-text optimized content</div>
{% else %}
  <div>HTML content</div>
{% endif %}
```

`previous_component_instance` and `next_component_instance` allow components to react to their position. Example — extra spacing when following a specific component:

```html
{% if render_context.previous_component_instance.component_name == 'hero' %}
  <div style="padding-top: 40px;">
{% else %}
  <div style="padding-top: 20px;">
{% endif %}
  <h1>{{headline}}</h1>
</div>
```

## ESP Code in Templates

Some ESPs use Liquid or similar templating syntax which can conflict with EmailShepherd's rendering. Use Liquid's `raw` tag to pass ESP code through without processing:

```liquid
{% raw %}Hello, {{ first_name }}{% endraw %}
```

This will be exported as:

```html
Hello, {{ first_name }}
```

Generally it's better to use EmailShepherd's personalization tags and conditionals to reference ESP code, but `raw` tags are useful when you need to embed ESP code directly in the template.

## Rules

- **Each template is isolated** — you can only reference fields defined within the same component, or global fields (from the container component).
- **Component `name` is a stable identifier** — changing it breaks existing emails that use the component. Changing `label` is always safe.
- **Field `liquid_variable` is a stable identifier** — renaming it loses saved values in existing emails.
- **Container template must include `{{children}}`**.
- Use `visible_if` to hide fields that are irrelevant based on other field values.
- Group related fields with the `group` property.
- Use design tokens for values that should be consistent but not editable per-component.
- Use semantic, descriptive names for `liquid_variable` values.
- Refer to `node_modules/@emailshepherd/eds-sdk/dist/types.d.ts` for exact type definitions.
