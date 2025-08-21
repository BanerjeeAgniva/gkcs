# Dark-Mode *Any* Website — Handy Guide (.md)

Want quick dark mode on sites that don’t support it? Here are reliable, copy-pasteable methods—from a 5-second DevTools hack to one-click bookmarklets, user styles, and power-user scripts. Pick what fits your workflow.

---

## TL;DR (fastest options)

* **DevTools (temporary):**
  Open Developer Tools → select `<body>` → **Styles** → add:

  ```css
  filter: invert(1) hue-rotate(180deg);
  background: #111 !important;
  ```

  Then “un-invert” media (add anywhere in Styles):

  ```css
  img, video, picture, svg, canvas, iframe { filter: invert(1) hue-rotate(180deg) !important; }
  ```

* **Bookmarklet (one-click toggle):**
  Create a bookmark → name it “Dark Mode Toggle” → paste this as the URL:

  ```text
  javascript:(function(){const css='html{filter:invert(1) hue-rotate(180deg)!important;background:#111!important} img,video,picture,svg,canvas,iframe{filter:invert(1) hue-rotate(180deg)!important} *{background-color:transparent!important}';const id='__instant_dark_mode';let s=document.getElementById(id);if(s){s.remove()}else{s=document.createElement('style');s.id=id;s.appendChild(document.createTextNode(css));document.documentElement.appendChild(s)}})();
  ```

---

## Why invert + hue-rotate?

* `invert(1)` flips light ↔ dark.
* `hue-rotate(180deg)` corrects weird color casts after inversion.
* We **re-apply** the same filter to media (images/videos/SVGs) so they look normal.

---

## Option 1: DevTools (quick + per-tab)

1. Open DevTools (F12 or `Ctrl+Shift+I` / `Cmd+Opt+I`).
2. Select the `<html>` or `<body>` element.
3. In **Styles**, paste:

   ```css
   filter: invert(1) hue-rotate(180deg);
   background: #111 !important;
   ```
4. Add the “un-invert” fix:

   ```css
   img, video, picture, svg, canvas, iframe {
     filter: invert(1) hue-rotate(180deg) !important;
   }
   ```

**Note:** Changes vanish on refresh. Use a bookmarklet or user style for persistence.

---

## Option 2: Bookmarklet (persistent, one click)

1. Create a new bookmark.
2. Set the **URL** to the JavaScript snippet in **TL;DR** above.
3. Click it on any page to toggle dark mode on/off.

---

## Option 3: Stylus user-style (per-site or global, persistent)

Install the **Stylus** extension (Chrome/Firefox). Create a new style:

```css
/* ==UserStyle==
@name         Instant Dark Mode (Invert + Fix Media)
@namespace    user
@version      1.0.0
@description  Darken any site by inverting with media fixes
@preprocessor default
==/UserStyle== */

@-moz-document domain("example.com") { /* change to "regexp(\".*\")" for all sites */
  html {
    filter: invert(1) hue-rotate(180deg) !important;
    background: #111 !important;
  }
  img, video, picture, svg, canvas, iframe {
    filter: invert(1) hue-rotate(180deg) !important;
  }
  /* Optional: tone down glaring backgrounds */
  * { background-color: transparent !important; }
}
```

* Scope to a single site by replacing `example.com`, or make it global.

---

## Option 4: Dark Reader (best results, minimal breakage)

Install the **Dark Reader** extension (Chrome/Firefox/Edge). It:

* Applies smart dark theming (not just inversion),
* Lets you toggle per-site,
* Adjusts contrast/brightness,
* Whitelists/blacklists sites.

**Recommended** if you want a set-and-forget experience.

---

## Option 5: Userscript (Tampermonkey/Greasemonkey)

For fine-grained control (auto-apply on chosen sites):

```javascript
// ==UserScript==
// @name         Instant Dark Mode (Invert + Fix Media)
// @namespace    user
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function () {
  const css = `
    html { filter: invert(1) hue-rotate(180deg) !important; background: #111 !important; }
    img, video, picture, svg, canvas, iframe { filter: invert(1) hue-rotate(180deg) !important; }
    * { background-color: transparent !important; }
  `;
  const id = '__instant_dark_mode';
  if (!document.getElementById(id)) {
    const s = document.createElement('style');
    s.id = id;
    s.appendChild(document.createTextNode(css));
    document.documentElement.appendChild(s);
  }
})();
```

* Adjust `@match` lines to target specific domains.

---

## For Site Owners/Developers (the “proper” way)

Add native dark mode using `prefers-color-scheme`:

```css
/* Light (default) */
:root {
  --bg: #ffffff;
  --fg: #111111;
}

/* Dark */
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0f1115;
    --fg: #e6e6e6;
  }
}

html, body { background: var(--bg); color: var(--fg); }
```

* Map components to CSS variables for colors.
* Provide brand-color tokens for both themes.
* Test contrast (aim WCAG AA or better).

---

## Tips, Caveats & Troubleshooting

* **Color-critical sites** (design tools, product images) may look off with inversion—use Dark Reader or native themes there.
* **Performance:** Full-page filters can be GPU-heavy on very large pages; user-style/extension approaches are usually fine.
* **Screenshots/Screen sharing:** You may capture the inverted look.
* **Over-dark elements:** You can selectively exclude containers:

  ```css
  .no-dark, .chart, .map { filter: none !important; background: #fff !important; color: #000 !important; }
  ```
* **Forms & inputs:** If placeholders become low-contrast, tweak:

  ```css
  input, textarea, select { background: #111 !important; color: #eee !important; }
  ::placeholder { color: #9aa0a6 !important; }
  ```

---

## Quick Checklist

* Need a **temporary** dark mode? → **DevTools** or **Bookmarklet**.
* Want **persistent** per-site control? → **Stylus** user-style.
* Want **smart**, low-maintenance dark mode? → **Dark Reader**.
* Building your **own site**? → Implement `prefers-color-scheme`.

---

*Copy this `.md` into your docs and tweak the snippets as needed. Happy dark-moding!* 🌙


<img width="1912" height="943" alt="image" src="https://github.com/user-attachments/assets/e0245e77-84ac-491c-a6ea-b56681b9a725" />
