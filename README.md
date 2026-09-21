# Segmented Toggle Switch Plugin — Oracle APEX

**By Hassaan Ahmed Tahir — Senior Oracle APEX Developer | [Read the full tutorial on ApexCodex](https://apexcodex.blogspot.com/2026/08/how-to-build-stylish-auto-login-toggle.html)**

Lightweight, fully configurable Oracle APEX **Item Plugin** that renders an LOV-driven, N-segment sliding toggle switch — a modern alternative to a radio group or select list for a small, fixed set of choices.

Instead of the flat list of radio buttons or a plain select list APEX gives you by default, this plugin turns any 2+ value LOV into a pill-shaped, animated switch: a sliding knob glides between segments, clicks trigger a short ripple effect, and the whole thing scales automatically with APEX's standard item-size classes. Colors — background, knob, and border — are fully configurable per item, so it drops straight into any theme.

> ✅ **End result:** A fully reusable, LOV-driven segmented toggle — configurable colors and border, smooth knob animation, click ripple feedback, and full keyboard accessibility — all controlled from the page designer, zero extra plumbing.

---

## 📦 What's in this repo

| File | Description |
|---|---|
| `toggle_switch.sql` | PL/SQL render procedure only — the source code registered in the plugin's Source section |
| `toggle_switch.css` | Stylesheet — track, sliding knob, segments, focus ring, click ripple animation |
| `toggle_switch.js` | Client-side behavior — positions the knob, handles clicks/keyboard, dispatches the change event |
| `README.md` | This guide — how the plugin is structured and built, step by step |

> ⚠️ **Note:** These three files are for reading and reuse. To actually install the plugin into your own APEX workspace, build it manually by following the steps below and pasting each file's content into the matching section of the plugin definition — Source, Files (CSS), and Files (JavaScript).

---

## 🚀 Quick install

1. In your APEX app, go to **App Builder → Shared Components → Plug-ins → Create**
2. Follow the steps below to configure the plugin's Name, Callbacks, Attributes, and Files sections
3. Paste `toggle_switch.sql` into the plugin's **Source**
4. Upload `toggle_switch.css` and `toggle_switch.js` under **Files**
5. Create a page item, set its **Type** to `Toggle switch`, define its List of Values, configure the colors you want, and run the page

---

## 📑 Table of Contents

1. [Create the Plugin (Name, Internal Name, Type)](#1-create-the-plugin-name-internal-name-type)
2. [Add the PL/SQL Source Code](#2-add-the-plsql-source-code)
3. [Set the Callback — Render Procedure Name](#3-set-the-callback--render-procedure-name)
4. [Configure "Supported For"](#4-configure-supported-for)
5. [Check the Standard Attributes](#5-check-the-standard-attributes)
6. [Create the Attribute Groups First](#6-create-the-attribute-groups-first)
7. [Set Up the Custom Attributes](#7-set-up-the-custom-attributes)
8. [Add the CSS File](#8-add-the-css-file)
9. [Add the JavaScript File](#9-add-the-javascript-file)
10. [Link Both Files Under "File URLs to Load"](#10-link-both-files-under-file-urls-to-load)
11. [Apply Changes to the Plugin](#11-apply-changes-to-the-plugin)
12. [Create the Page Item and Test It](#12-create-the-page-item-and-test-it)

---

## 1. Create the Plugin (Name, Internal Name, Type)

Head to **Shared Components → Plug-ins → Create** and fill in the Name section exactly as shown below:

| Field | Value |
|---|---|
| Name | Toggle switch |
| Internal Name | TOGGLE_SWITCH |
| Type | Item |

---

## 2. Add the PL/SQL Source Code

In the **Source** section, paste the full contents of [`toggle_switch.sql`](./toggle_switch.sql).

This procedure runs the item's own **List of Values** query through `apex_plugin_util.get_data` to fetch display/return value pairs, prints the standard hidden input APEX uses to track the item's session state, then loops over the LOV rows to render one `<span class="sh-tgl-segment">` per value — marking whichever one matches the item's current value as active. The background, knob, and border colors and thickness are written inline via `style="..."` from the plugin's own custom attributes, so no color needs to be hardcoded in the CSS.

> 💡 **Tip:** This plugin requires the item's **List of Values** to be configured with exactly two columns (display value, return value) — the render procedure calls `apex_plugin_util.get_data` with `p_min_columns => 2` and `p_max_columns => 2`, so an LOV with a different column count will raise an error at render time.

---

## 3. Set the Callback — Render Procedure Name

Scroll to the **Callbacks** section and enter the render procedure's name so APEX knows which procedure to call whenever this plugin needs to be drawn:

| Field | Value |
|---|---|
| Render Procedure/Function Name | `render_toggle_switch` |

Meta Data, AJAX, and Validation Procedure fields stay empty — the switch's value is written straight into APEX's standard hidden input and session state, exactly like a native radio group or select list, so no extra server round trip is needed.

---

## 4. Configure "Supported For"

Under **Supported for**, tick both Component Types:

- ☑️ Page Items
- ☑️ Interactive Grid Columns

---

## 5. Check the Standard Attributes

In **Standard Attributes**, enable the checkboxes below. Since this plugin is LOV-driven and renders its own colored track/knob element (not a native input), it needs **Has List of Values** switched on and marked Required, plus the color/width attributes a themeable component like this needs:

| ✅ Enabled | ✅ Enabled |
|---|---|
| Is Visible Widget | Standard Form Element |
| Session State Changeable | Has Source Attributes |
| Has Element Attributes | Has Width Attributes |
| Has Element Option Attribute | Has Icon Attribute |
| Has Encrypt Session State Attribute | Has List of Values |

Under **Standard Attribute additional Meta Data**, mark **Has List of Values** as **Required** — this is what forces a developer to define the LOV before the item will render.

Everything else (Has Placeholder Attribute, Has Filter, Has Link, Has Cascading LOV Attributes, Format Mask, etc.) stays unchecked — this isn't a text-entry or filterable item, it's a fixed-choice switch.

---

## 6. Create the Attribute Groups First

With colors and border settings to configure, group them before adding a single custom attribute so they land in a logical, collapsible section in the page designer:

| Title | Sequence |
|---|---|
| Color Settings | 1 |
| Border Settings | 2 |

> 💡 **Tip:** Groups are created once, up front, precisely so that when you add each custom attribute in the next step you simply pick its group from a dropdown instead of reorganizing things afterward.

---

## 7. Set Up the Custom Attributes

Now add all 4 Custom Attributes, each mapped to its `p_item.attribute_0N` value and assigned to the group it belongs to. All four are plain **Color** or **Number** attributes with no dependency on one another — the switch always renders with either the developer's chosen colors or the render procedure's own sensible defaults:

| Label | Scope | Group | Seq | Attr # | Type | Required | Default | Depending on |
|---|---|---|---|---|---|---|---|---|
| Background Color | Component | Color Settings | 10 | 1 | Color | No | — | — |
| Toggle Color | Component | Color Settings | 20 | 2 | Color | No | — | — |
| Border Color | Component | Border Settings | 30 | 3 | Color | No | — | — |
| Border Thickness | Component | Border Settings | 40 | 4 | Number | No | — | — |

> ✅ **Why defaults live in PL/SQL, not the attribute definition:** Each color attribute is read with `NVL(p_item.attribute_0N, '<fallback>')` in the render procedure (`#eef0f3` for the track, `#3a6df0` for the knob, `rgba(0,0,0,0.08)` for the border, `1` for thickness) — so a developer can leave every one of these four attributes blank and still get a fully styled, good-looking switch out of the box.

---

## 8. Add the CSS File

Under **Files**, create `toggle_switch.css` and paste the contents of [`toggle_switch.css`](./toggle_switch.css) from this repo.

This stylesheet handles the pill-shaped outer track (`.sh-tgl-toggle`), the absolutely-positioned sliding knob (`.sh-tgl-knob`) that animates its `transform` and `width` with a bouncy cubic-bezier easing, the segment labels and their hover/active states, a keyboard-only focus ring (shown only when `.sh-tgl-using-keyboard` is set by the JS), responsive sizing driven by APEX's standard `.t-Form-fieldContainer--large` / `--xlarge` / `--stretchInputs` classes, and the short-lived click-ripple animation.

> 💡 **Tip:** Border color and width are deliberately left undefined in the CSS (`border-style: solid` only) — they're set inline per item from the plugin's own custom attributes in the PL/SQL, so the same stylesheet works correctly no matter what color a developer picks in the page designer.

---

## 9. Add the JavaScript File

Now create `toggle_switch.js` and paste the contents of [`toggle_switch.js`](./toggle_switch.js) from this repo.

This is the heart of the plugin — namespaced as `shToggleSwitch` to avoid global scope conflicts, with defensive `null` checks on every DOM lookup so a missing element never throws an unhandled error that could block APEX's onload queue.

**Core capabilities of the script:**
- Reads the segment count passed in from the render procedure's onload call and calculates each segment's width as a percentage
- Finds whichever segment PL/SQL already marked active and slides the knob under it on load — falling back to the first segment (and writing its value into the hidden input) if nothing matched
- Tracks keyboard vs. mouse navigation so the focus ring only appears for keyboard users, not mouse clicks
- On click: updates the hidden input's value, moves the active class and the knob, spawns a short-lived ripple effect at the exact click position, and dispatches a native `change` event so APEX session state and Dynamic Actions pick up the new value
- On keyboard: `Enter` and `Space` trigger the focused segment exactly like a click, for full accessibility

---

## 10. Link Both Files Under "File URLs to Load"

Scroll to **File URLs to Load** and reference both minified files using APEX's plugin file substitution syntax — APEX automatically generates and serves the `#MIN#` versions once the files are uploaded:

| Field | Value |
|---|---|
| Cascading Style Sheet | `#PLUGIN_FILES#style#MIN#.css` |
| JavaScript | `#PLUGIN_FILES#javascript#MIN#.js` |

---

## 11. Apply Changes to the Plugin

Click **Apply Changes** — the plugin is now fully registered and ready to be attached to any page item in the application.

> ✅ We're finished with the plugin itself. Everything from here is about testing it on a real page.

---

## 12. Create the Page Item and Test It

On your page, create a new page item:

| Field | Value |
|---|---|
| Name | (your item name) |
| Type | Toggle switch |

Define a **List of Values** with exactly two columns — display value first, return value second — for example a static LOV like `STATIC:Active;A,Inactive;I` or a simple `SELECT display_value, return_value FROM ...` query. Then, in the item's attribute groups, optionally set *Background Color*, *Toggle Color*, *Border Color*, and *Border Thickness* to match your theme — or leave them blank to use the plugin's built-in defaults. Save and run the page.

The switch renders as a pill-shaped track with one segment per LOV row and the knob resting under whichever value is currently active. Click any segment (or tab to it and press Enter/Space) — the knob glides smoothly to that segment, a short ripple animates outward from the click point, and the item's session state and any attached Dynamic Actions update immediately via the native `change` event.

> 💡 **Tip:** The switch's width and height automatically scale up when the page item's own **Item Width** attribute is set to Large or X-Large (via APEX's standard `.t-Form-fieldContainer--large` / `--xlarge` classes) — no extra configuration needed on the plugin's own attributes.

---

## Why This Approach Works So Well

- **Fully LOV-driven** — works with any 2+ segment list of values, static or dynamic, so the same plugin covers a 2-way Yes/No switch and a 5-way status selector alike
- **Themeable without touching CSS** — background, knob, and border colors (plus border thickness) are all page-designer attributes, with sensible defaults baked into the render procedure
- **Native session-state integration** — writes into APEX's own hidden input and dispatches a real `change` event, so Dynamic Actions and session state behave exactly like a native radio group or select list
- **Polished interaction out of the box** — a bouncy sliding knob animation and a per-click ripple effect, with zero extra configuration
- **Fully accessible** — every segment is keyboard-focusable and operable with Enter/Space, with a focus ring that only appears for keyboard users
- **Defensive by design** — every DOM lookup in the JavaScript is null-checked, so a missing or malformed element logs a console warning instead of breaking APEX's page-load sequence

---

That's the full build — from an empty plugin shell to a production-ready, fully themeable, LOV-driven segmented toggle switch. If you build this into your own app, feel free to open an issue or share which color combinations you end up using most!

🙏 **A quick thanks** — this plugin wouldn't be where it is without **Sajjad Hanifa**, who's been a huge help shaping these features and pushing for building them right. Really appreciate the support!

📌 **Stay tuned** — more Oracle APEX plugins and tutorials are coming. Read the original write-up for this plugin on [ApexCodex](https://apexcodex.blogspot.com/2026/08/how-to-build-stylish-auto-login-toggle.html).
