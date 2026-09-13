# 🎨 Shopify Custom Font Integration
## Complete Beginner's Guide — Any Shopify Theme

### *"Zero Shopify Knowledge" Edition*

![Shopify](https://img.shields.io/badge/Shopify-Any%20Theme-96BF48?style=for-the-badge&logo=shopify&logoColor=white)
![Liquid](https://img.shields.io/badge/Liquid-Template%20Language-blue?style=for-the-badge&logo=shopify)

> **💡 Zero Shopify experience needed.**
> Every click, every menu name, and every button is spelled out — guiding you through **building this feature from scratch**, in the exact order you'd actually build it.

---

## 📋 Table of Contents

1. [Beginner Glossary — Words You'll See Everywhere](#1-beginner-glossary--words-youll-see-everywhere)
2. [What You're Building — Quick Overview](#2-what-youre-building--quick-overview)
3. [How to Open the Code Editor](#3-how-to-open-the-code-editor)
4. [Building It — Step by Step](#4-building-it--step-by-step)
   - [Step 1 — Add Typography Settings (`settings_schema.json`)](#step-1--add-typography-settings-settings_schemajson)
   - [Step 2 — Create `snippets/fonts.liquid`](#step-2--create-snippetsfontsliquid)
   - [Step 3 — Add Font Variables to `snippets/css-variables.liquid`](#step-3--add-font-variables-to-snippetscss-variablesliquid)
   - [Step 4 — Render the Snippets in `layout/theme.liquid`](#step-4--render-the-snippets-in-layoutthemeliquid)
5. [How `fonts.liquid` Works — Line by Line](#5-how-fontsliquid-works--line-by-line)
6. [How `css-variables.liquid` Works — Line by Line](#6-how-css-variablesliquid-works--line-by-line)
7. [How `settings_schema.json` Works — Field Types Explained](#7-how-settings_schemajson-works--field-types-explained)
8. [Using It — Method 1: Shopify Font Picker](#8-using-it--method-1-shopify-font-picker)
9. [Using It — Method 2: Google Fonts](#9-using-it--method-2-google-fonts)
10. [Using It — Method 3: Custom .woff2 Upload](#10-using-it--method-3-custom-woff2-upload)
11. [Using the Font Variables in Your CSS](#11-using-the-font-variables-in-your-css)
12. [How to Check If It Actually Worked](#12-how-to-check-if-it-actually-worked)
13. [Common Mistakes & Fixes](#13-common-mistakes--fixes)
14. [Quick Checklist](#14-quick-checklist)

---

## 1. Beginner Glossary — Words You'll See Everywhere

Before anything else, here's what the confusing words actually mean. Keep this section open in another tab.

| Word | What it really means |
|---|---|
| **Theme** | The full set of files that controls how your store looks. Like a "template" for your whole shop. |
| **Liquid** | Shopify's own coding language (mix of HTML + simple logic like `{% if %}`). Files ending in `.liquid` use it. |
| **Snippet** | A small, reusable chunk of Liquid code stored in its own file, kept inside the `snippets` folder. You "call" it from another file instead of repeating the same code everywhere. |
| **Section** | A bigger, drag-and-drop block (like "Hero banner" or "Featured products") that merchants can add/remove/reorder in the Customizer. Different from a snippet — sections show up in the visual editor, snippets don't. |
| **Layout (`theme.liquid`)** | The master file that wraps every single page of your store (it has the `<html>`, `<head>`, `<body>` tags). Almost everything else gets loaded inside this file. |
| **`{% render 'something' %}`** | The command that says "go open the file `snippets/something.liquid` and place its output right here." This is how a snippet gets "attached" to a page. |
| **Customizer** | The drag-and-drop visual page editor in Shopify Admin (`Online Store → Themes → Customize`). Merchants use this — no code. |
| **Edit code** | The raw code editor inside Shopify Admin where developers (you!) directly edit `.liquid`, `.json`, and `.css` files. |
| **`settings_schema.json`** | A JSON file that defines what options show up inside **Theme Settings** in the Customizer (toggles, dropdowns, color pickers, etc.) |
| **CSS variable** (e.g. `--font-heading`) | A reusable value you define once (usually in `:root`) and then reuse anywhere in your CSS with `var(--font-heading)`. Change it in one place, it updates everywhere. |
| **CDN URL** | The public web address Shopify gives a file after you upload it (e.g. `cdn.shopify.com/...`). Anyone can load the file from this link. |
| **`.woff2`** | A compressed web font file format — the modern standard for loading custom fonts on websites (smaller & faster than older formats). |
| **`font-display: swap`** | A browser instruction that says "show fallback text immediately, then swap to the custom font once it loads" — prevents invisible text while fonts load. |

---

## 2. What You're Building — Quick Overview

By the end of this guide, your theme will have a complete, merchant-friendly font system: 4 independent font roles (Heading, Subheading, Body, Accent), each switchable between Shopify's built-in font library, a Google Fonts link, or an uploaded `.woff2` file — all controlled from the Customizer, no code touching needed after setup.

Here's what you'll create or edit, and in what order:

```
your-theme/
├── layout/
│   └── theme.liquid              ✏️  STEP 4 — edit: add 2 render lines
├── snippets/
│   ├── fonts.liquid               🆕 STEP 2 — create: loads the font files
│   └── css-variables.liquid       ✏️  STEP 3 — edit: add --font-heading etc.
└── config/
    └── settings_schema.json       ✏️  STEP 1 — edit: add the Typography section
```

> 📌 Build order matters here: the **settings** need to exist first (Step 1), because both `fonts.liquid` and `css-variables.liquid` read values from those settings. Then the snippets are created/edited (Steps 2–3). Only at the very end do you "switch it on" by rendering the snippets inside `theme.liquid` (Step 4).

---

## 3. How to Open the Code Editor

1. Log into your **Shopify Admin** (`your-store.myshopify.com/admin`)
2. In the left sidebar, click **Online Store**
3. Click **Themes**
4. Find your live or draft theme → click the **`…`** (three dots) button next to it
5. Click **Edit code**

You'll land on a screen with a file tree on the left (folders like `Layout`, `Sections`, `Snippets`, `Templates`, `Config`, `Assets`, `Locales`) and a code editor on the right.

> 🧭 Alternative route: `Online Store → Themes → Customize → ` then click the small **pencil/code icon (`</>`)** at the bottom-left of the Customizer sidebar — it also opens **Edit code**.

---

## 4. Building It — Step by Step

### Step 1 — Add Typography Settings (`settings_schema.json`)

This file controls what fields show up under **Theme settings → Typography** in the Customizer. We add the settings **first**, because Steps 2 and 3 will read values from these setting IDs (`heading_custom_enable`, `heading_font`, etc.) — they won't work without this in place.

### Where to find/open it

1. **Edit code** (Section 3) → in the left file tree, click **Config** → click `settings_schema.json`
2. This file is one big JSON array `[ ... ]`, where each item is a settings section (General, Colors, Typography, Cart, etc.)
3. Find a sensible spot — usually right after an existing section like **Colors** — and add a **comma** after that section's closing `}`, then paste the block below as a new item

> ⚠️ **JSON rule to remember:** every item in the array needs a comma **after** it, **except the very last item**. A comma after the last item is invalid JSON and can break your theme save/deploy. Double-check this after pasting.

### Full code to add

```json
{
  "name": "t:general.typography",
  "settings": [
    {
      "type": "header",
      "content": "Heading"
    },
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
    },
    {
      "type": "select",
      "id": "heading_text_case",
      "label": "Text case",
      "options": [
        { "value": "default", "label": "Default" },
        { "value": "capitalize", "label": "Capitalize" },
        { "value": "uppercase", "label": "Uppercase" }
      ],
      "default": "capitalize"
    },
    {
      "type": "range",
      "id": "h1_font_size",
      "label": "H1 font size",
      "min": 28,
      "max": 80,
      "step": 2,
      "unit": "px",
      "default": 48
    },
    {
      "type": "header",
      "content": "Subheading"
    },
    {
      "type": "font_picker",
      "id": "subheading_font",
      "label": "Font",
      "default": "work_sans_n4",
      "visible_if": "{{ settings.subheading_custom_enable == false }}"
    },
    {
      "type": "checkbox",
      "id": "subheading_custom_enable",
      "label": "Use custom font",
      "default": false
    },
    {
      "type": "text",
      "id": "subheading_custom_family",
      "label": "Font name",
      "default": "Custom subheading",
      "visible_if": "{{ settings.subheading_custom_enable }}"
    },
    {
      "type": "url",
      "id": "subheading_custom_url",
      "label": "Font url",
      "visible_if": "{{ settings.subheading_custom_enable }}",
      "info": "Paste a direct .woff2 font file URL or a Google Fonts CSS link."
    },
    {
      "type": "select",
      "id": "subheading_custom_style",
      "label": "subheading Font Style",
      "options": [
        { "value": "normal", "label": "Normal" },
        { "value": "italic", "label": "Italic" }
      ],
      "default": "normal",
      "visible_if": "{{ settings.subheading_custom_enable }}"
    },
    {
      "type": "range",
      "id": "subheading_custom_weight",
      "label": "subheading Font Weight",
      "min": 100,
      "max": 1000,
      "step": 100,
      "default": 400,
      "visible_if": "{{ settings.subheading_custom_enable }}"
    },
    {
      "type": "select",
      "id": "subheading_text_case",
      "label": "Text case",
      "options": [
        { "value": "default", "label": "Default" },
        { "value": "capitalize", "label": "Capitalize" },
        { "value": "uppercase", "label": "Uppercase" }
      ],
      "default": "capitalize"
    },
    {
      "type": "header",
      "content": "Body"
    },
    {
      "type": "font_picker",
      "id": "body_font",
      "label": "Font",
      "default": "work_sans_n4",
      "visible_if": "{{ settings.body_custom_enable == false }}"
    },
    {
      "type": "checkbox",
      "id": "body_custom_enable",
      "label": "Use custom font",
      "default": false
    },
    {
      "type": "text",
      "id": "body_custom_family",
      "label": "Font name",
      "default": "Custom Body",
      "visible_if": "{{ settings.body_custom_enable }}"
    },
    {
      "type": "url",
      "id": "body_custom_url",
      "label": "Font url",
      "visible_if": "{{ settings.body_custom_enable }}",
      "info": "Paste a direct .woff2 font file URL or a Google Fonts CSS link."
    },
    {
      "type": "select",
      "id": "body_custom_style",
      "label": "body Font Style",
      "options": [
        { "value": "normal", "label": "Normal" },
        { "value": "italic", "label": "Italic" }
      ],
      "default": "normal",
      "visible_if": "{{ settings.body_custom_enable }}"
    },
    {
      "type": "range",
      "id": "body_custom_weight",
      "label": "body Font Weight",
      "min": 100,
      "max": 1000,
      "step": 100,
      "default": 400,
      "visible_if": "{{ settings.body_custom_enable }}"
    },
    {
      "type": "header",
      "content": "Accent"
    },
    {
      "type": "font_picker",
      "id": "accent_font",
      "label": "Font",
      "default": "work_sans_n4",
      "visible_if": "{{ settings.accent_custom_enable == false }}"
    },
    {
      "type": "checkbox",
      "id": "accent_custom_enable",
      "label": "Use custom font",
      "default": false
    },
    {
      "type": "text",
      "id": "accent_custom_family",
      "label": "Font name",
      "default": "Custom Accent",
      "visible_if": "{{ settings.accent_custom_enable }}"
    },
    {
      "type": "url",
      "id": "accent_custom_url",
      "label": "Font url",
      "visible_if": "{{ settings.accent_custom_enable }}",
      "info": "Paste a direct .woff2 font file URL or a Google Fonts CSS link."
    },
    {
      "type": "select",
      "id": "accent_custom_style",
      "label": "accent Font Style",
      "options": [
        { "value": "normal", "label": "Normal" },
        { "value": "italic", "label": "Italic" }
      ],
      "default": "normal",
      "visible_if": "{{ settings.accent_custom_enable }}"
    },
    {
      "type": "range",
      "id": "accent_custom_weight",
      "label": "accent Font Weight",
      "min": 100,
      "max": 1000,
      "step": 100,
      "default": 400,
      "visible_if": "{{ settings.accent_custom_enable }}"
    }
  ]
},
```

> ✅ Notice the very last setting (`accent_custom_weight`) has **no comma** after its closing `}` — only the closing `}` of the whole Typography block has a trailing comma, because more sections likely follow it in the file. If this happens to be the **last** section in your file, remove that trailing comma too.

4. Click **Save**

---

### Step 2 — Create `snippets/fonts.liquid`

This snippet reads the settings from Step 1 and actually loads the correct font (Shopify picker, Google Fonts link, or uploaded `.woff2`) depending on what's selected.

### How to create the file

1. **Edit code** → in the left file tree, find the **Snippets** section
2. Click **`+ Add a new snippet`** (or the **`…`** menu next to "Snippets" → **Add a new snippet**)
3. In the popup, type the name **without** the extension: `fonts` → Shopify creates `snippets/fonts.liquid`
4. Click **Create snippet**
5. Delete any placeholder content, then paste the code below
6. Click **Save**

### Full code to paste

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

---

### Step 3 — Add Font Variables to `snippets/css-variables.liquid`

This snippet turns the settings into CSS variables (`--font-heading`, `--font-body`, etc.) that the rest of your theme's CSS can use.

### If the file doesn't exist yet

1. **Edit code** → **Snippets** → **`+ Add a new snippet`** → name it `css-variables` → **Create snippet**

### If the file already exists (common — it's often also used for color variables)

1. Open `snippets/css-variables.liquid` from the file tree
2. Find the existing `:root { ... }` block
3. Paste the font lines below **inside** that same `:root { }` — don't create a second `:root` block, just add these lines alongside whatever's already there (colors, spacing, etc.)

### Full code (font-related part)

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

> 📝 If your file already has its own `{% style %} :root { ... } {% endstyle %}` wrapper (for colors, for example), just copy the lines **between** `:root {` and `}` above — don't duplicate the `{% style %}` / `:root` / `{% endstyle %}` wrappers themselves.

3. Click **Save**

---

### Step 4 — Render the Snippets in `layout/theme.liquid`

The settings exist (Step 1) and the snippets exist (Steps 2–3) — but nothing actually runs until you tell the layout file to render them. This is the final "switch on" step.

### How to do it

1. **Edit code** → in the left file tree, click **Layout** → click `theme.liquid`
2. Find the `<head>` section near the top of the file
3. Add these two lines **inside** `<head>...</head>`:

```liquid
{% render 'fonts' %}
{% render 'css-variables' %}
```

> ⚠️ **Order matters.** `fonts` must render **before** `css-variables`, because the font files need to start loading before the CSS that references them. If you already have other renders in `<head>` (like `meta-tags`, `color-schemes`, `stylesheets`), a sensible placement looks like this:

```liquid
<head>
  {% render 'meta-tags' %}
  {% render 'fonts' %}
  {%- render 'color-schemes' -%}
  {% render 'css-variables' %}
  {%- render 'stylesheets' -%}
  {%- render 'scripts' -%}

  {{ content_for_header }}
</head>
```

4. Click **Save**

✅ At this point, the whole system is wired up end to end. Open **Customize** in the Shopify Admin and you should see a new **Typography** section under **Theme settings**.

---

### ⚠️ Don't Forget — Also add to `layout/password.liquid`

> **Why?** Shopify's `password.liquid` is a **completely separate, standalone layout file** — it is **not** a page that loads inside `theme.liquid`. This means any `{% render %}` calls you added in `theme.liquid` are **completely invisible** to the password page. If you skip this step, the password page will fall back to a generic system font, ignoring all your font settings entirely.

1. **Edit code** → click **Layout** → click `password.liquid`
2. Find the `<head>` section (it's a short file — usually 20–40 lines)
3. Add the **exact same two lines** inside `<head>...</head>`:

```liquid
{% render 'fonts' %}
{% render 'css-variables' %}

4. Click **Save**

> 📝 The `:root { }` CSS variables set by `css-variables.liquid` are now available on the password page too, so any CSS you write there using `var(--font-heading)` etc. will work correctly.

---

## 5. How `fonts.liquid` Works — Line by Line

**Block 1 — builds the list of 4 font "roles":**
```liquid
{%- assign fonts = 'heading,subheading,body,accent' | split: ',' -%}
```
This turns the text `"heading,subheading,body,accent"` into a list: `[heading, subheading, body, accent]`. The rest of the file loops over this list 4 times — once per role — instead of writing the same code 4 separate times.

**Block 2 — injects the Google Fonts `<link>` tag (Method 2 only):**
```liquid
{% if enable and url contains 'fonts.googleapis.com' %}
  <link rel="stylesheet" href="{{ url }}">
{% endif %}
```
For each role, it checks: is "Use custom font" turned ON, **and** does the URL you pasted contain `fonts.googleapis.com`? If both are true, it prints a real `<link>` tag into the page's `<head>` — this is the same tag Google Fonts gave you, just inserted automatically.

**Block 3 — loads Shopify's built-in fonts (Method 1 only):**
```liquid
{% unless custom_enable %}
  {{ font_picker | font_face: font_display: 'swap' }}
  ...
{% endunless %}
```
`unless custom_enable` means "only run this when custom font is OFF." The `font_face` filter is a special Shopify Liquid filter that automatically writes the correct `@font-face` CSS for whatever font you picked in the dropdown — including separate rules for bold and italic versions.

**Block 4 — builds `@font-face` for your uploaded `.woff2` file (Method 3 only):**
```liquid
{% if enable and url != blank and url contains ".woff" %}
  @font-face{
    font-family:"{{ family }}";
    src:url("{{ url }}") format("woff2");
    font-display:swap;
  }
{% endif %}
```
This only runs if: custom font is ON, **and** a URL was actually entered, **and** that URL contains `.woff` (so it correctly skips Google Fonts URLs and only reacts to uploaded font files). It writes a standard CSS `@font-face` rule using the **Font name** you typed as the `font-family`.

**`{% style %}...{% endstyle %}`** — this Shopify tag automatically wraps everything inside it in a real `<style>` HTML tag when the page loads, so Blocks 3 & 4 (which write CSS) end up correctly placed inside `<head>`.

---

## 6. How `css-variables.liquid` Works — Line by Line

Using Heading as the example — the same pattern repeats for all 4 roles:

```liquid
{% if settings.heading_custom_enable %}
  --font-heading: "{{ settings.heading_custom_family }}", sans-serif;
```
"If the merchant turned ON custom font for Heading, set the CSS variable `--font-heading` to whatever text they typed into the Font name field." The quotes (`"..."`) around the value matter — without them, a font name with a space in it (like `Playfair Display`) would break the CSS.

```liquid
{% else %}
  --font-heading: {{ settings.heading_font.family }}, {{ settings.heading_font.fallback_families }};
```
"Otherwise (custom font is OFF), pull the family name and a safe fallback font straight from Shopify's font picker object." `fallback_families` is something Shopify generates automatically — you never set it yourself.

```liquid
--heading-text-case: {{ settings.heading_text_case }};
--font-h1-size: {{ settings.h1_font_size }}px;
```
These two lines sit **outside** the if/else — meaning they always run, no matter which method you're using. They come from separate, independent settings.

> 📝 **Note:** only **Heading** and **Subheading** have a `text_case` (Default / Capitalize / Uppercase) option, and only **Heading** has the `h1_font_size` slider. **Body** and **Accent** don't have these extra controls in the Step 1 schema above — that's intentional, not a bug.

---

## 7. How `settings_schema.json` Works — Field Types Explained

| `"type"` value | What it draws in the Customizer |
|---|---|
| `font_picker` | A searchable dropdown of Shopify's built-in fonts |
| `checkbox` | A simple ON/OFF toggle |
| `text` | A single-line text input box |
| `url` | A text input box, but Shopify validates it looks like a real URL |
| `select` | A dropdown with fixed choices you define in `"options"` |
| `range` | A draggable slider between a min and max |

### The magic behind `"visible_if"`

```json
"visible_if": "{{ settings.heading_custom_enable }}"
```
This single line is **why fields appear and disappear** as you click the checkbox. Shopify reads this like a mini Liquid `{% if %}` — "only show this field in the Customizer if `heading_custom_enable` is true." This is exactly how Method 1's fields hide when you switch to Method 2/3, and vice versa.

---

## 8. Using It — Method 1: Shopify Font Picker

Best for: quick changes, no copy-pasting URLs, Shopify's built-in font library (1000+ fonts).

1. **Shopify Admin** → **Online Store** → **Themes**
2. Click **Customize** on your live/draft theme
3. In the bottom-left of the Customizer sidebar, click the **Theme settings** icon (looks like a small paint roller / gear)
4. Click **Typography**
5. Expand a group (**Heading / Subheading / Body / Accent**)
6. Make sure **"Use custom font"** is **OFF**
7. Click the **Font** dropdown → browse/search → pick a font
8. Repeat for the other roles if needed → **Save**

---

## 9. Using It — Method 2: Google Fonts

### Step 1 — Get the Google Fonts URL

1. Go to [fonts.google.com](https://fonts.google.com)
2. Search and click a font (e.g. **Playfair Display**)
3. Select the weights you need (e.g. tick **400** Regular and **700** Bold)
4. Click **Get embed code** (panel slides in from the right)
5. Click the **`<link>` tab** (not `@import`)
6. You'll see something like:
   ```html
   <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap" rel="stylesheet">
   ```
7. **Copy only the part inside `href="..."`**:
   ```
   https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap
   ```

> ❓ Why only the `<link>` tab? Because `fonts.liquid` builds the `<link>` tag itself using your URL — it expects a plain link address, not an `@import` CSS rule.

### Step 2 — Paste it into the Customizer

1. **Online Store → Themes → Customize → Theme settings → Typography**
2. Expand the font role → turn **"Use custom font"** ✅ ON
3. Fill in:
   - **Font name:** `Playfair Display` (must match exactly, including capitalization)
   - **Font url:** paste the URL you copied
   - **Font Style:** `Normal` or `Italic`
   - **Font Weight:** e.g. `700` (must be a weight you actually selected on Google Fonts)
4. **Save**

---

## 10. Using It — Method 3: Custom .woff2 Upload

### Step 1 — Upload your `.woff2` files

| File you have | What it's for |
|---|---|
| `MyBrand-Regular.woff2` | Regular weight — **required**, this is the only one you'll paste a URL for |
| `MyBrand-Bold.woff2` | Bold weight — upload only if you use bold text |
| `MyBrand-Italic.woff2` | Italic — upload only if you use italics |
| `MyBrand-BoldItalic.woff2` | Bold + Italic — upload only if you use both together |

1. **Shopify Admin** → **Content** → **Files**
2. Click **Upload files** → select your `.woff2` files
3. Click **Copy link** next to the **Regular** file specifically
4. You'll get a URL like:
   ```
   https://cdn.shopify.com/s/files/1/XXXX/XXXX/files/MyBrand-Regular.woff2
   ```

> 💡 You only ever paste the **Regular** file's URL into the Customizer. Bold/italic files don't need a setting — the browser finds them once `fonts.liquid` generates the `@font-face` rule, as long as they're uploaded too.

### Step 2 — Paste it into the Customizer

1. **Online Store → Themes → Customize → Theme settings → Typography**
2. Expand the font role → turn **"Use custom font"** ✅ ON
3. Fill in:
   - **Font name:** `MyBrand` (any name you choose — becomes the `font-family` value)
   - **Font url:** paste the Regular file's CDN URL
   - **Font Style:** `Normal`
   - **Font Weight:** `400`
4. **Save**

---

## 11. Using the Font Variables in Your CSS

Once `css-variables.liquid` runs, every visitor's browser receives this in the page's `<head>`:

```css
:root {
  --font-heading: "Playfair Display", sans-serif;
  --font-heading-style: normal;
  --font-heading-weight: 700;
  --heading-text-case: capitalize;
  --font-h1-size: 48px;

  --font-subheading: Work Sans, sans-serif;
  --font-body: "DM Sans", sans-serif;
  --font-accent: "Inter", sans-serif;
}
```

Anywhere in the theme's CSS (usually `assets/base.css`), reuse these instead of hardcoding font names:

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

button, .btn {
  font-family: var(--font-accent);
}
```

**Why bother with variables instead of just typing `font-family: "Playfair Display"` everywhere?** Because if a merchant later changes the heading font in the Customizer, it updates in **one place** (`:root`) — every CSS rule using `var(--font-heading)` updates automatically, with zero extra code changes.

---

## 12. How to Check If It Actually Worked

1. **Preview your store** (the eye icon in the Customizer, or visit your storefront URL)
2. Right-click anywhere → **Inspect** (opens browser DevTools)
3. Go to **Console**, paste this, press Enter:
   ```javascript
   getComputedStyle(document.body).getPropertyValue('--font-heading')
   ```
   You should see your font name printed back, e.g. `"Playfair Display", sans-serif`
4. Go to **Network** tab → reload → filter by **Font** (or search `.woff`) → confirm status **200** (not 404)
5. Go to **Elements** tab → expand `<head>` → for Google Fonts, confirm a `<link href="https://fonts.googleapis.com/...">` tag is present

If all 3 checks pass, the font is correctly wired up.

---

## 13. Common Mistakes & Fixes

### ❌ Font isn't showing, fallback font shows instead
- Double-check **"Use custom font"** is actually ON for that specific role
- The **Font name** must match **exactly**, including capitalization
- Open your Google Fonts URL directly in a new browser tab — if it doesn't load there, it won't load on your site either

### ❌ Google Fonts URL doesn't work
```
❌ Wrong:  <link href="https://fonts.googleapis.com/css2?family=..." />
✅ Right:  https://fonts.googleapis.com/css2?family=...
```
Paste **only the bare URL** — not the full `<link>` HTML tag.

### ❌ `.woff2` file gives a 404 error
- Confirm the file is actually uploaded: **Content → Files**
- Re-copy the link directly from the Files page
- Confirm the URL ends in `.woff2`, not `.woff`

### ❌ Bold text looks too thin / not actually bold
- For Google Fonts: make sure your URL includes the bold weight, e.g. `wght@400;700`
- The **Font Weight** slider value must be a weight your font file actually supports

### ❌ CSS variable is set correctly but the font still isn't applying
```css
.your-element {
  font-family: var(--font-heading) !important;
}
```
Then check **DevTools → Elements → (select the element) → Computed tab → search `font-family`** to see which rule is winning.

### ❌ Customizer didn't save / theme errored out
- Almost always a JSON syntax mistake in `settings_schema.json` — usually a missing or extra comma. Re-check Step 1 carefully against the code block provided.

---

## 14. Quick Checklist

### Setup (one-time):
- [ ] Step 1: Typography block added to `config/settings_schema.json` — no trailing/missing commas
- [ ] Step 2: `snippets/fonts.liquid` created with the full code
- [ ] Step 3: Font variables added inside `:root { }` in `snippets/css-variables.liquid`
- [ ] Step 4: `{% render 'fonts' %}` and `{% render 'css-variables' %}` added to `<head>` in `layout/theme.liquid`, in that order
- [ ] Step 4 (password page): Same `{% render 'fonts' %}` and `{% render 'css-variables' %}` lines also added to `<head>` in `layout/password.liquid` — `password.liquid` is a **separate standalone layout** and does **not** inherit renders from `theme.liquid`
- [ ] Customizer → Theme settings → **Typography** section appears

### Google Fonts route:
- [ ] fonts.google.com → pick font + weights → "Get embed code" → `<link>` tab → copy only the `href` URL
- [ ] Customizer → Typography → role → "Use custom font" ✅ ON → fill fields → Save

### Custom .woff2 route:
- [ ] Admin → Content → Files → upload all variants → copy the **Regular** file's CDN link
- [ ] Customizer → "Use custom font" ✅ ON → fill fields → Save

### Always verify:
- [ ] Browser Console: `getComputedStyle(document.body).getPropertyValue('--font-heading')` returns the right font
- [ ] Network tab → font file request returns status **200**
- [ ] Elements → `<head>` → Google Fonts `<link>` tag present (Method 2 only)

---

## 💡 Pro Tips

1. **Speed:** `&display=swap` in the Google Fonts URL combined with the snippet's built-in `font-display: swap` keeps text visible while fonts load — never remove it.
2. **Don't over-load:** only select the exact weights you'll actually use on Google Fonts (e.g. just `400` and `700`) — extra weights = extra load time for nothing.
3. **Always test in Incognito/Private mode** after a font change, so a cached old font doesn't fool you into thinking it didn't work.
4. **Variable fonts:** a single modern `.woff2` "variable font" file can cover multiple weights at once — ask your font provider if a variable version exists before uploading 4 separate files.
5. To reuse this exact system on a different theme, just repeat Steps 1–4 there — it's fully self-contained.

---

*Beginner-friendly, build-from-scratch guide for the Skeleton theme's custom font system — June 2026.*

---

## 📜 License & Usage

> ⚠️ **You are free to use this code in your own Shopify projects.**
> However, you **cannot** claim ownership, re-publish it as your own work, or remove the author credit.

| ✅ You CAN | ❌ You CANNOT |
|---|---|
| Copy & use this code in your store | Claim this guide or code as your own |
| Modify it to fit your theme | Re-publish it without giving credit |
| Share this guide with others | Remove the author's name or links |
| Learn from it freely | Sell it as a product without permission |

> This project is shared in the spirit of open learning — **use it, learn from it, build with it** — just keep the credit where it belongs. 🙏

---

## 👤 Author

**Made with ❤️ by Harshank Patel**

[![GitHub](https://img.shields.io/badge/GitHub-HarshankPatel9038-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/HarshankPatel9038)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-patel--harshank-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/patel-harshank/)

*If this guide helped you, consider giving the repo a ⭐ — it means a lot!*
