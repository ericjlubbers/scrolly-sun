# scrolly-sun — GSAP Demo Roadmap

A plan for adding GSAP scroll animations to a single Gutenberg article on WordPress,
loaded exclusively on that page via Perfmatters, with a clear path to expand later.

---

## Stack & Constraints

- **WordPress**: Managed/hosted (WP Engine or similar) — no direct file access
- **Script loading**: [Perfmatters](https://perfmatters.io/) Code Snippets manager
- **GSAP tier**: Free (public CDN via jsDelivr)
- **Plugins required**: GSAP core + ScrollTrigger (no additional plugins)
- **Build step**: None — all JS is plain, inline code managed via Perfmatters

---

## Phase 1 — Build the Article in Gutenberg

1. Create or open the target article in the WordPress editor.
2. **Note the Post ID** — visible in the URL when editing: `post=XXXX`. You'll need this for Conditions targeting in Phase 2.
3. For each block you want to animate, open **Inspector → Advanced → Additional CSS class(es)** and assign a semantic class:

   | Class | Purpose |
   |---|---|
   | `anim-hero` | Hero / title area (on-load entrance) |
   | `anim-section` | Content sections (scroll-triggered) |
   | `anim-card` | Cards, pull quotes, callouts (interactive) |

   These are your GSAP selectors — no JS changes needed per block.

---

## Phase 2 — Load GSAP via Perfmatters (scoped to this article only)

Navigate to **Perfmatters → Code → New Snippet** and create **Snippet A**.

**Snippet A: "GSAP CDN Loader"**

| Setting | Value |
|---|---|
| Type | HTML |
| Location | Frontend footer |
| Conditions → Include | Post ID = `XXXX` |

```html
<script src="https://cdn.jsdelivr.net/npm/gsap@3.13/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.13/dist/ScrollTrigger.min.js"></script>
```

> Loading synchronously in the footer ensures GSAP is defined before the animation
> snippet runs. The Conditions rule locks both scripts to one post — zero impact on
> the rest of the site.

---

## Phase 3 — Animation JS via Perfmatters

Create **Snippet B** in the same Code manager.

**Snippet B: "GSAP Article Animations"**

| Setting | Value |
|---|---|
| Type | JS |
| Location | Frontend footer |
| Priority | `20` (runs after Snippet A at default priority `10`) |
| Conditions → Include | Post ID = `XXXX` (same post as Snippet A) |
| Print method | Inline (demo); switch to File later for edge caching |

```js
document.addEventListener('DOMContentLoaded', function () {
  gsap.registerPlugin(ScrollTrigger);

  // Respect reduced-motion preference
  var mm = gsap.matchMedia();

  mm.add('(prefers-reduced-motion: no-preference)', function () {

    // On-load entrance — hero
    gsap.from('.anim-hero', {
      opacity: 0,
      y: 40,
      duration: 1,
      ease: 'power2.out'
    });

    // Scroll-triggered sections
    gsap.utils.toArray('.anim-section').forEach(function (el) {
      gsap.from(el, {
        scrollTrigger: { trigger: el, start: 'top 85%' },
        opacity: 0,
        y: 30,
        duration: 0.7,
        ease: 'power2.out'
      });
    });

    // Interactive cards — hover scale
    document.querySelectorAll('.anim-card').forEach(function (card) {
      card.addEventListener('mouseenter', function () {
        gsap.to(card, { scale: 1.03, duration: 0.25, ease: 'power1.out' });
      });
      card.addEventListener('mouseleave', function () {
        gsap.to(card, { scale: 1, duration: 0.25, ease: 'power1.out' });
      });
    });

  });
});
```

---

## Phase 4 — Iteration Workflow

| Goal | Method |
|---|---|
| Adjust timing, easing, or offsets | Edit Snippet B in Perfmatters Code manager, save, reload the page |
| Prototype a new tween without saving | Run `gsap.to('.anim-section', {...})` in browser DevTools console |
| Slow all animations for inspection | `gsap.globalTimeline.timeScale(0.1)` in DevTools console |
| Fine-tune scroll trigger start/end points | Temporarily add `markers: true` inside `scrollTrigger: {}`, then remove |
| Recover from a broken snippet | Perfmatters auto-deactivates on fatal JS errors; or open **Code → Settings → Enable Safe Mode** |

---

## Expanding to Other Pages

When ready to roll out beyond the demo article, update the Conditions rule in
**both snippets**:

- Change Include from **Post ID = XXXX** → **Post Type = Posts** (or any broader scope)
- No other changes needed — the snippet code is already production-ready

Optionally, switch Snippet B's print method from **Inline** to **File** for longer
CDN/edge cache lifetimes on the animation JS.

---

## Verification Checklist

- [ ] Article loads with no JS errors in DevTools console
- [ ] `gsap` is defined in console (type `gsap` — returns the GSAP object)
- [ ] `ScrollTrigger` is registered (`gsap.plugins.scrollTrigger` is defined)
- [ ] `.anim-hero` fades/rises in on page load
- [ ] `.anim-section` elements animate as they enter the viewport on scroll
- [ ] `.anim-card` elements scale on hover
- [ ] A different post / the homepage: GSAP CDN scripts are **not** present in Network tab
- [ ] DevTools → Rendering → "Emulate CSS prefers-reduced-motion: reduce" → animations do not play

---

## References

- [GSAP Docs](https://gsap.com/docs/v3/)
- [ScrollTrigger Docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger)
- [Perfmatters Code Snippets](https://perfmatters.io/docs/code-snippets/)
- [Perfmatters Conditions Builder](https://perfmatters.io/docs/code-snippets/#conditions)
