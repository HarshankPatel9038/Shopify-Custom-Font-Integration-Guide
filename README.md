# 🎨 Shopify Custom Font Integration Guide
### Skeleton Theme — Beginner-Friendly Complete Guide

---

## 📋 Table of Contents

1. [Font System Overview](#1-font-system-overview)
2. [Theme Font Architecture](#2-theme-font-architecture)
3. [snippets/fonts.liquid — Full Code + Explanation](#3-fontsliquid)
4. [snippets/css-variables.liquid — Full Code + Explanation](#4-css-variablesliquid)
5. [Method 1: Shopify Font Picker](#5-method-1-shopify-font-picker)
6. [Method 2: Google Fonts](#6-method-2-google-fonts)
7. [Method 3: Custom .woff2 Upload](#7-method-3-woff2-upload)
8. [Theme Customizer Settings (settings_schema.json)](#8-theme-customizer-settings)
9. [CSS Variables — How to Use in CSS](#9-css-variables-use)
10. [Common Mistakes & Fixes](#10-common-mistakes)
11. [Quick Checklist](#11-quick-checklist)

---

## 1. Font System Overview {#1-font-system-overview}

This theme supports **4 font roles** — each can be customized independently:

| Role | CSS Variable | Used For |
|------|-------------|----------|
| `heading` | `--font-heading` | H1, H2, H3 headings |
| `subheading` | `--font-subheading` | Subheadings, labels |
| `body` | `--font-body` | Paragraph text, descriptions |
| `accent` | `--font-accent` | Buttons, inputs |

Each role supports **3 methods** for loading fonts:

- ✅ **Shopify Font Picker** — Built-in, easiest
- ✅ **Google Fonts URL** — Free, large library
- ✅ **Custom .woff2 File** — Brand or purchased fonts

---

## 2. Theme Font Architecture {#2-theme-font-architecture}

```
layout/theme.liquid
├── {% render 'fonts' %}         ← Google Fonts <link> + @font-face load
├── {% render 'css-variables' %} ← CSS variables (:root) set
└── {% render 'stylesheets' %}   ← base.css load
```

```
snippets/
├── fonts.liquid          ← Font loading logic (Section 3)
└── css-variables.liquid  ← CSS custom properties (Section 4)

config/
└── settings_schema.json  ← Customizer UI settings
```

> ⚠️ Always render `fonts` **before** `css-variables` — fonts must load first.

---

## 3. snippets/fonts.liquid — Full Code + Explanation {#3-fontsliquid}

**Create `snippets/fonts.liquid` and paste this code:**

```liquid
{%- assign fonts = 'heading,subheading,body,accent' | split: ',' -%}

{%- comment -%}
  Load Google fonts when custom font enabled
{%- endcomment -%}
{%- for font in fonts -%}
  {%- assign enable_key = font | append: '_custom_enable' -%}
  {%- assign url_key = font | append: '_custom_url' -%}
  {%- assign enable = settings[enable_key] -%}
  {%- assign url = settings[url_key] -%}
  {% if enable and url contains 'fonts.googleapis.com' %}
    <link rel="stylesheet" href="{{ url }}">
  {% endif %}
{%- endfor -%}

{% style %}
    {%- comment -%}
    Load Shopify font variants when custom font disabled
    {%- endcomment -%}
    {%- for font in fonts -%}

    {%- assign enable_key = font | append: "_custom_enable" -%}
    {%- assign font_key = font | append: "_font" -%}

    {%- assign custom_enable = settings[enable_key] -%}
    {%- assign font_picker = settings[font_key] -%}

    {% unless custom_enable %}
      {{ font_picker | font_face: font_display: 'swap' }}
      {{ font_picker | font_modify: 'weight', 'bold' | font_face: font_display: 'swap' }}
      {{ font_picker | font_modify: 'style', 'italic' | font_face: font_display: 'swap' }}
      {{ font_picker | font_modify: 'weight', 'bold' | font_modify: 'style', 'italic' | font_face: font_display: 'swap' }}
    {% endunless %}

  {%- endfor -%}


    {%- comment -%}
    Custom font files (.woff2)
    {%- endcomment -%}
     {%- for font in fonts -%}

    {%- assign enable_key = font | append: "_custom_enable" -%}
    {%- assign url_key = font | append: "_custom_url" -%}
    {%- assign family_key = font | append: "_custom_family" -%}

    {%- assign enable = settings[enable_key] -%}
    {%- assign url = settings[url_key] -%}
    {%- assign family = settings[family_key] -%}

    {% if enable and url != blank and url contains ".woff" %}
      @font-face{
        font-family:"{{ family }}";
        src:url("{{ url }}") format("woff2");
        font-display:swap;
      }
    {% endif %}

  {%- endfor -%}
{% endstyle %}
```

### Logic Breakdown

**Block 1 — Google Fonts `<link>` inject (inside `<head>`):**

```liquid
{%- assign fonts = 'heading,subheading,body,accent' | split: ',' -%}
```
→ Creates an array of 4 font roles: `['heading', 'subheading', 'body', 'accent']`

```liquid
{%- assign enable_key = font | append: '_custom_enable' -%}
{%- assign enable = settings[enable_key] -%}
```
→ Builds a dynamic key: `heading` → `heading_custom_enable` → reads Customizer checkbox value

```liquid
{% if enable and url contains 'fonts.googleapis.com' %}
  <link rel="stylesheet" href="{{ url }}">
{% endif %}
```
→ Custom ON + Google Fonts URL → injects `<link>` tag into `<head>`

---

**Block 2 — Shopify Font Picker (when custom is OFF):**

```liquid
{% unless custom_enable %}
  {{ font_picker | font_face: font_display: 'swap' }}
  {{ font_picker | font_modify: 'weight', 'bold' | font_face: font_display: 'swap' }}
  {{ font_picker | font_modify: 'style', 'italic' | font_face: font_display: 'swap' }}
  {{ font_picker | font_modify: 'weight', 'bold' | font_modify: 'style', 'italic' | font_face: font_display: 'swap' }}
{% endunless %}
```
→ `font_face` filter — Shopify auto-generates `@font-face` CSS
→ Loads all 4 variants: Regular, Bold, Italic, Bold Italic
→ `unless custom_enable` — only runs when custom font is OFF

---

**Block 3 — Custom .woff2 `@font-face` (when custom is ON + .woff2 URL):**

```liquid
{% if enable and url != blank and url contains ".woff" %}
  @font-face{
    font-family:"{{ family }}";
    src:url("{{ url }}") format("woff2");
    font-display:swap;
  }
{% endif %}
```
→ `url contains ".woff"` — automatically excludes Google Fonts URLs, only processes .woff2 files
→ `family` → value from Customizer "Font name" field
→ Inside `{% style %}...{% endstyle %}` — Shopify wraps output in a `<style>` tag

---

## 4. snippets/css-variables.liquid — Full Code + Explanation {#4-css-variablesliquid}

**Create `snippets/css-variables.liquid` and paste this code:**

```liquid
{% style %}
  :root {
    /* Heading Font */
    {% if settings.heading_custom_enable %}
      --font-heading: "{{ settings.heading_custom_family }}", sans-serif;
      --font-heading-style: {{ settings.heading_custom_style }};
      --font-heading-weight: {{ settings.heading_custom_weight }};
    {% else %}
      --font-heading: {{ settings.heading_font.family }}, {{ settings.heading_font.fallback_families }};
      --font-heading-style: {{ settings.heading_font.style }};
      --font-heading-weight: {{ settings.heading_font.weight }};
    {% endif %}
    --heading-text-case: {{ settings.heading_text_case }};
    --font-h1-size: {{ settings.h1_font_size }}px;

    /* Subheading Font */
    {% if settings.subheading_custom_enable %}
      --font-subheading: "{{ settings.subheading_custom_family }}", sans-serif;
      --font-subheading-style: {{ settings.subheading_custom_style }};
      --font-subheading-weight: {{ settings.subheading_custom_weight }};
    {% else %}
      --font-subheading: {{ settings.subheading_font.family }}, {{ settings.subheading_font.fallback_families }};
      --font-subheading-style: {{ settings.subheading_font.style }};
      --font-subheading-weight: {{ settings.subheading_font.weight }};
    {% endif %}
    --subheading-text-case: {{ settings.subheading_text_case }};

    /* Body Font */
    {% if settings.body_custom_enable %}
      --font-body: "{{ settings.body_custom_family }}", sans-serif;
      --font-body-style: {{ settings.body_custom_style }};
      --font-body-weight: {{ settings.body_custom_weight }};
    {% else %}
      --font-body: {{ settings.body_font.family }}, {{ settings.body_font.fallback_families }};
      --font-body-style: {{ settings.body_font.style }};
      --font-body-weight: {{ settings.body_font.weight }};
    {% endif %}

    /* Accent Font */
    {% if settings.accent_custom_enable %}
      --font-accent: "{{ settings.accent_custom_family }}", sans-serif;
      --font-accent-style: {{ settings.accent_custom_style }};
      --font-accent-weight: {{ settings.accent_custom_weight }};
    {% else %}
      --font-accent: {{ settings.accent_font.family }}, {{ settings.accent_font.fallback_families }};
      --font-accent-style: {{ settings.accent_font.style }};
      --font-accent-weight: {{ settings.accent_font.weight }};
    {% endif %}
  }
{% endstyle %}
```

### Logic Breakdown

Same if/else pattern for every font role — `heading` as example:

```liquid
{% if settings.heading_custom_enable %}
  --font-heading: "{{ settings.heading_custom_family }}", sans-serif;
  --font-heading-style: {{ settings.heading_custom_style }};
  --font-heading-weight: {{ settings.heading_custom_weight }};
```
→ Custom ON → uses Customizer "Font name" field value as CSS variable
→ Quotes are important — needed for multi-word font names (e.g., `Playfair Display`)

```liquid
{% else %}
  --font-heading: {{ settings.heading_font.family }}, {{ settings.heading_font.fallback_families }};
  --font-heading-style: {{ settings.heading_font.style }};
  --font-heading-weight: {{ settings.heading_font.weight }};
{% endif %}
```
→ Custom OFF → uses Shopify font picker object properties directly
→ `fallback_families` — Shopify auto-generates safe fallback (e.g., `sans-serif`)

```liquid
--heading-text-case: {{ settings.heading_text_case }};
--font-h1-size: {{ settings.h1_font_size }}px;
```
→ These variables are independent of custom enable/disable — always set

---

## 5. Method 1: Shopify Font Picker {#5-method-1-shopify-font-picker}

**Easiest option — use Shopify's built-in font library**

### Steps:
1. **Online Store** → **Themes** → **Customize**
2. **Theme Settings** → **Typography**
3. Select a font role (Heading / Subheading / Body / Accent)
4. Keep **"Use custom font"** checkbox **OFF**
5. Pick a font from the dropdown
6. **Save**

### What happens in the background:

`fonts.liquid` — runs the `unless custom_enable` block:
```liquid
{{ font_picker | font_face: font_display: 'swap' }}
```

`css-variables.liquid` — runs the `else` block:
```liquid
--font-heading: {{ settings.heading_font.family }}, {{ settings.heading_font.fallback_families }};
```

---

## 6. Method 2: Google Fonts {#6-method-2-google-fonts}

### Step 1 — Get the Google Fonts URL:

1. Go to [fonts.google.com](https://fonts.google.com)
2. Select a font (e.g., **Playfair Display**)
3. Choose weights (400, 700 recommended)
4. Click **"Get embed code"**
5. Select the **`<link>`** tab — **not** `@import` (that's for CSS files)
6. Copy only the URL from inside `href="..."`

```html
<!-- You'll see this tag — copy only the href value -->
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap" rel="stylesheet">
```

```
✅ Paste this URL in the Customizer:
https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap
```

> 💡 **Why the `<link>` tab?** `fonts.liquid` generates `<link rel="stylesheet" href="{{ url }}">` internally — so the href URL format is required.

### Step 2 — Set in Customizer:

1. **Theme Settings** → **Typography** → font role
2. **"Use custom font"** ✅ ON
3. Fill in:
   - **Font name:** `Playfair Display`
   - **Font URL:** the URL from above
   - **Font Style:** Normal
   - **Font Weight:** 700
4. **Save**

### What happens in the background:

`fonts.liquid` — injects Google Fonts link:
```liquid
{% if enable and url contains 'fonts.googleapis.com' %}
  <link rel="stylesheet" href="{{ url }}">
{% endif %}
```

`css-variables.liquid` — runs the `if` block:
```liquid
--font-heading: "Playfair Display", sans-serif;
```

---

## 7. Method 3: Custom .woff2 Upload {#7-method-3-woff2-upload}

### Step 1 — Upload .woff2 Files

Upload all the font variants you want to use in the theme:

| File | Variant |
|------|---------|
| `MyBrand-Regular.woff2` | Regular (400) — **required** |
| `MyBrand-Bold.woff2` | Bold (700) — upload if bold is used |
| `MyBrand-Italic.woff2` | Italic (400 italic) — upload if italic is used |
| `MyBrand-BoldItalic.woff2` | Bold Italic (700 italic) — upload if both are used |

Upload steps:
1. Shopify Admin → **Content** → **Files**
2. Click **Upload files** → upload each `.woff2` file
3. After upload, click **Copy link** next to each file → note the Regular file URL:
   ```
   https://cdn.shopify.com/s/files/1/XXXX/XXXX/files/MyBrand-Regular.woff2
   ```

> 💡 Only the **Regular** file URL needs to be pasted in the Customizer. `fonts.liquid` will auto-generate the `@font-face` block — Bold and Italic variants are handled automatically by the browser using the uploaded files.

### Step 2 — Set Regular URL in Customizer

1. **Theme Settings** → **Typography** → font role (e.g., Heading)
2. **"Use custom font"** ✅ ON
3. Fill in:
   - **Font name:** `MyBrand`
   - **Font URL:** CDN URL of the **Regular** file only
   - **Font Style:** `Normal`
   - **Font Weight:** `400`
4. **Save**

`fonts.liquid` will auto-generate this `@font-face`:

```css
@font-face {
  font-family: "MyBrand";
  src: url("https://cdn.shopify.com/.../MyBrand-Regular.woff2") format("woff2");
  font-display: swap;
}
```

---

## 8. Theme Customizer Settings {#8-theme-customizer-settings}

**`config/settings_schema.json`** — Heading font block:

```json
{
  "type": "font_picker",
  "id": "heading_font",
  "label": "Font",
  "default": "work_sans_n4",
  "visible_if": "{{ settings.heading_custom_enable == false }}"
},
{
  "type": "checkbox",
  "id": "heading_custom_enable",
  "label": "Use custom font",
  "default": false
},
{
  "type": "text",
  "id": "heading_custom_family",
  "label": "Font name",
  "default": "Custom Heading",
  "visible_if": "{{ settings.heading_custom_enable }}"
},
{
  "type": "url",
  "id": "heading_custom_url",
  "label": "Font url",
  "visible_if": "{{ settings.heading_custom_enable }}",
  "info": "Paste a direct .woff2 font file URL or a Google Fonts CSS link."
},
{
  "type": "select",
  "id": "heading_custom_style",
  "label": "Heading Font Style",
  "options": [
    { "value": "normal", "label": "Normal" },
    { "value": "italic", "label": "Italic" }
  ],
  "default": "normal",
  "visible_if": "{{ settings.heading_custom_enable }}"
},
{
  "type": "range",
  "id": "heading_custom_weight",
  "label": "Heading Font Weight",
  "min": 100,
  "max": 1000,
  "step": 100,
  "default": 400,
  "visible_if": "{{ settings.heading_custom_enable }}"
}
```

> 🔁 Same pattern for `subheading`, `body`, and `accent` — just replace the role name.

---

## 9. CSS Variables — How to Use in CSS {#9-css-variables-use}

`css-variables.liquid` outputs `:root` CSS variables in the browser:

```css
:root {
  --font-heading: "Playfair Display", sans-serif;
  --font-heading-style: normal;
  --font-heading-weight: 700;
  --heading-text-case: none;
  --font-h1-size: 48px;

  --font-subheading: Work Sans, sans-serif;
  --font-subheading-style: normal;
  --font-subheading-weight: 400;
  --subheading-text-case: none;

  --font-body: "DM Sans", sans-serif;
  --font-body-style: normal;
  --font-body-weight: 400;

  --font-accent: "Inter", sans-serif;
  --font-accent-style: normal;
  --font-accent-weight: 500;
}
```

### How to use in sections / base.css:

```css
h1, h2, h3 {
  font-family: var(--font-heading);
  font-weight: var(--font-heading-weight);
  font-style: var(--font-heading-style);
  text-transform: var(--heading-text-case);
}

p, li, span {
  font-family: var(--font-body);
  font-weight: var(--font-body-weight);
}

.subheading {
  font-family: var(--font-subheading);
  font-weight: var(--font-subheading-weight);
}

button, .btn {
  font-family: var(--font-accent);
}
```

### Verify in Browser Console:

```javascript
getComputedStyle(document.body).getPropertyValue('--font-heading')
// Output: "Playfair Display", sans-serif
```

---

## 10. Common Mistakes & Fixes {#10-common-mistakes}

### ❌ Font not loading
**Symptom:** Font name is set but fallback font is showing

**Fix:**
- Is `heading_custom_enable` checkbox ON? Check
- Font name must exactly match — `Playfair Display` (case sensitive)
- Open the Google Fonts URL in browser to test if it's valid

### ❌ Google Fonts URL error
```
Wrong: <link href="https://fonts.googleapis.com/css2?family=..." />
Right: https://fonts.googleapis.com/css2?family=...
```
**Fix:** Paste only the URL — not the HTML tag

### ❌ .woff2 Font 404 Error
**Fix:**
- Confirm file is uploaded in Shopify Admin → Content → Files
- Is the CDN URL copied correctly?
- URL must end in `.woff2` — `.woff` ≠ `.woff2`

### ❌ Font weight looks wrong (bold text appears thin)
**Fix:**
- Include the required weight in the Google Fonts URL: `wght@400;700`
- Customizer Weight slider must match what the font file supports

### ❌ CSS variable is set but font is not applying
```css
.your-element {
  font-family: var(--font-heading) !important;
}
```
Check in Browser DevTools → Element → Computed → `font-family`

---

## 11. Quick Checklist {#11-quick-checklist}

### Google Fonts:
- [ ] fonts.google.com → select font + weights
- [ ] "Get embed code" → `<link>` tab → copy href URL only
- [ ] Customizer → Typography → "Use custom font" ✅ ON
- [ ] Fill Font name exactly as shown on Google Fonts
- [ ] Paste URL → Save → Preview

### Custom .woff2:
- [ ] Shopify Admin → Content → Files → upload .woff2 files
- [ ] Copy CDN URL of the Regular file
- [ ] Customizer → "Use custom font" ✅ ON
- [ ] Fill Font name + Regular URL → Save → Preview

### Browser Verification:
- [ ] Console: `getComputedStyle(document.body).getPropertyValue('--font-heading')`
- [ ] Network tab → font file returns 200 status (not 404)
- [ ] Elements → `<head>` → `<link>` tag is present (Google Fonts)

---

## 💡 Pro Tips

1. **Performance:** `&display=swap` in the Google Fonts URL — `font-display: swap` is applied automatically
2. **Safe fallback:** When custom is OFF, Shopify font picker provides automatic fallback
3. **Cache:** Test in incognito mode to avoid browser cache issues
4. **Weights:** Only load the weights you need in the Google Fonts URL — avoid extras
5. **Variable Fonts:** A single `.woff2` variable font file can support multiple weights

---

*Guide prepared based on Skeleton Theme code review — June 2026*
