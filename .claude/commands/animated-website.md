---
description: 7-step pipeline that builds a complete animated single-page website from a text brief using UI UX Pro Max, Stitch, Magic, Firecrawl, and Nano Banana 2.
argument-hint: [optional brief]
---

# Animated Website Builder Pipeline

You are orchestrating a 7-step pipeline that turns a short brief into a complete animated single-page website.

If the user invoked this command with no arguments, your very first message must be exactly:

> Animated website builder pipeline is ready. What would you like to build?

Then wait for the brief. If arguments were passed, treat them as the brief and skip the greeting.

After receiving the brief, execute the pipeline below, narrating each step briefly (one sentence per step) before running its tool calls. For every optional tool, if it is unavailable or fails, apply the fallback and keep moving — never halt the pipeline on an optional failure.

---

## Step 1 — Intake & Plan

Extract from the brief: product/topic, audience, tone, primary CTA, any reference URLs, any brand colors/fonts the user named, and preferred style keywords. If critical info is missing, ask one consolidated question; otherwise proceed with reasonable defaults.

Produce a short plan (sections, palette direction, typography direction, motion language). Show the plan to the user and ask for confirmation before continuing. If the user says "go" or similar, continue without re-confirming later steps.

## Step 2 — Design System (UI UX Pro Max — REQUIRED)

Invoke the `ui-ux-pro-max` skill to lock in:
- a design style (from the 50+ styles)
- a color palette (from the 161 palettes)
- a font pairing (from the 57 pairings)

Record the chosen tokens (hex values, font family names, radii, spacing scale, motion timings). These tokens are the single source of truth for all later steps.

## Step 3 — Reference Scrape (Firecrawl — OPTIONAL)

Only if the brief included a reference URL:
- Call `firecrawl_scrape` to pull the page's structure, copy tone, and hero layout.
- Summarize what to borrow (layout rhythm, headline voice) vs. what to avoid.

Fallback: skip silently if no URL was given or the MCP is unavailable.

## Step 4 — Screen Layouts (Stitch — OPTIONAL)

If the `stitch` MCP is green:
- Call `create_project` → `generate_screen_from_text` for each section (hero, features, social proof, pricing/CTA, footer) using the locked design tokens from Step 2.
- Call `fetch_screen_code` to retrieve the generated React/HTML per screen.

Fallback: if Stitch is unavailable, hand-author section layouts directly in the final HTML in Step 7 using the Step 2 tokens.

## Step 5 — Hero & Feature Imagery (Nano Banana 2 — OPTIONAL)

If `infsh` is authenticated:
- Generate the hero image and any feature illustrations via:
  `infsh app run google/gemini-3-1-flash-image-preview --input '{"prompt": "...", "aspect_ratio": "16:9"}'`
- Save returned URLs/paths and reference them in the HTML.

Fallback: generate inline SVG illustrations that match the palette (gradient blobs, abstract shapes, simple icons).

## Step 6 — Polished Components (Magic / 21st.dev — OPTIONAL)

If the user's brief specifically asked for a 21st.dev component (navbar, pricing table, testimonial grid, bento, etc.) AND the `magic` MCP is green:
- Call `21st_magic_component_builder` with the request plus the Step 2 tokens.
- Splice the returned component into the corresponding section.

Fallback: hand-code the component using Tailwind + the Step 2 tokens.

## Step 7 — Assemble & Animate (REQUIRED)

Produce a single `index.html` at the project root that:
- Loads Tailwind (CDN is fine for a one-file deliverable) and the chosen Google Fonts.
- Loads Framer Motion via its UMD build (or uses CSS `@keyframes` + `IntersectionObserver` if Motion is not desired).
- Applies the Step 2 tokens as CSS custom properties on `:root`.
- Composes Step 4's sections (or hand-authored fallbacks) top-to-bottom.
- Embeds Step 5's images (or SVG fallbacks).
- Adds motion: hero entrance (fade + rise), scroll-reveal on each section, subtle hover micro-interactions on CTAs, prefers-reduced-motion respected.
- Is responsive (mobile-first, breakpoints at 640 / 1024).
- Includes a real `<title>`, meta description, and OG tags derived from the brief.

After writing the file, print a short summary:
- what was built,
- which optional tools fired vs. fell back,
- how to preview (`python3 -m http.server 8000` then open `http://localhost:8000`).

Do NOT open a browser or start a server unless the user asks.

---

## Pipeline Rules

- Never stop the pipeline for an optional-tool failure — log it and fall back.
- Never invent design tokens outside Step 2's output.
- Keep tool calls parallel when steps are independent (e.g., Step 5 image generations).
- The final deliverable is one `index.html` unless the user asks for a multi-file build.

$ARGUMENTS
