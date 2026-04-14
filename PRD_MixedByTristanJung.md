# ⚠️ Always Do First

Invoke the `frontend-design` skill before writing any frontend code, every session, no exceptions.

---

# Project Requirements Document
## Mixed by Tristan Jung — Mix Engineer Portfolio Website

**Document Version:** 4.6  
**Prepared for:** Developer Handoff  
**Client:** Tristan Jung, Professional Mix Engineer  
**Domain:** `tristanjung.net`  
**Reference Draft:** Mixed_by_Tristan_Jung.html  

---

## 1. Project Overview

This document defines the requirements for building a two-page portfolio website for Tristan Jung, a professional mix engineer. The site balances portfolio showcase with professional credibility — presenting Tristan's work through audio-enabled album art, while providing professional context via modal overlays.

The site is built as two HTML files with embedded CSS and vanilla JavaScript. No frameworks, no build tools, no dependencies beyond a single self-hosted font file.

---

## 2. Target Audience

- Artists and labels seeking a mix engineer

---

## 3. Tech Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 (two files: `index.html` and `portfolio.html`) |
| Styling | CSS3 with custom properties (CSS variables) |
| Interactivity | Vanilla JavaScript (ES6+) |
| Fonts | Mansalva Regular — self-hosted TTF (`Media/Fonts/Mansalva-Regular.ttf`) |
| Hosting | Static hosting (e.g. Cloudflare Pages, Netlify, GitHub Pages) |
| Email obfuscation | Cloudflare email protection (optional, site-level) |

No JavaScript frameworks, no bundlers, no npm packages are required.

---

## 4. Visual Design System

### 4.1 Color Palette

| Token | Hex Value | Usage |
|---|---|---|
| `--primary-text` | `#3B2F2F` | All text site-wide (headings, body, nav, card text) |
| `--bg-primary` | `#8A9BA8` | Modal backgrounds, tooltip background |
| `--white` | `#ffffff` | General utility where pure white is needed |
| `--black` | `#000000` | General utility |

### 4.2 Typography

The site uses a single typeface site-wide: **Mansalva Regular**, loaded from a local TTF asset. No Google Fonts link is used.

**Font loading (both pages):**

```css
@font-face {
  font-family: 'Mansalva';
  src: url('Media/Fonts/Mansalva-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
```

All elements use `font-family: 'Mansalva', serif` as the base. Since only one weight is available, hierarchy is established through **size and letter-spacing variation**:

| Role | Size | Letter-spacing | Notes |
|---|---|---|---|
| Hero `<h1>` | `clamp(2.5rem, 8vw, 5rem)` | `-2px` | Page 1 only |
| Section heading (`<h2>`) | `clamp(2.5rem, 5vw, 4rem)` | `-1px` | "Selected Works" |
| Nav links | `2.376rem` desktop / `1.62rem` mobile | `0px` | About, Contact |
| Modal heading (`<h2>`) | `3rem` desktop / `2rem` mobile | `-1px` | — |
| Body / card metadata | `1rem` default | `0.02em` | Artist names, body copy |
| Email (contact) | `1.65rem` | `0px` | Contact modal + footer |

All `font-weight` values throughout the site are `400`.

### 4.3 Background Texture

Both pages use the **texture image as the direct background** of the entire site — there is no solid background color. The texture tiles across the full viewport on both pages.

**Implementation:**

```css
body {
  background-image: url('Media/Texture/texture.png');
  background-repeat: repeat;
  background-size: auto;
  background-attachment: fixed;
}
```

- `background-attachment: fixed` keeps the texture stationary as the page scrolls — the texture does not scroll with content.

**Asset note:** Client to provide the texture PNG before developer handoff. A seamless tile is required — the asset must repeat without visible seams. The texture should be dark enough on its own to provide sufficient contrast for `--primary-text` (`#3B2F2F`) without any color overlay.

---

### 4.4 Spacing Scale


Use CSS custom properties for all spacing to maintain consistency:

```
--spacing-xs:  4px
--spacing-sm:  8px
--spacing-md:  16px
--spacing-lg:  24px
--spacing-xl:  40px
--spacing-xxl: 64px
```

### 4.5 Border Radius Conventions

- Navigation links: `35px` (pill shape)
- Modal containers: `30px`
- Work card images: `20px`
- Work cards themselves: `20px`
- Modal close button: `50%` (circle)

---

## 5. Stacking Order (z-index Reference)

All z-index values are defined here as the single source of truth. No component should use a z-index value not listed in this table.

| z-index | Element | Notes |
|---|---|---|
| `1` | Base page content | Default stacking context for all sections |
| `2` | Carousel cards | All cards at the same level — no card lifts above another |
| `100` | Fixed navigation bar | Sits above page content; covered by modal backdrop |
| `200` | Modal backdrop (`rgba(0,0,0,0.8)`) | Covers everything including the nav bar |
| `201` | Modal card | Sits above its own backdrop |
| `9999` | Loading screen | Must be topmost on initial Page 1 load |

**Key behaviors this enforces:**
- The modal backdrop (z-index `200`) is higher than the nav (z-index `100`), so opening a modal fully covers the nav bar
- All carousel cards share z-index `2` — overlap appearance is handled by `transform: scale()` only, not stacking
- The texture is the raw `body` background — no pseudo-element or z-index needed

---

## 6. Site Architecture

The site consists of two distinct pages:

```
┌─────────────────────────────────────────────┐
│  PAGE 1 — index.html (Landing)              │
│                                             │
│  [Loading screen — auto-dismisses 1.5s]     │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │     "Mixed by Tristan Jung"  (h1)   │    │  ← Fullscreen, no scroll
│  │          click → Page 2            │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
                      ↓
              Page 2 slides up over Page 1
                      ↓
┌─────────────────────────────────────────────┐
│  PAGE 2 — portfolio.html (Portfolio)        │
│                                             │
│  [About]               [Contact]            │  ← Fixed nav, modals
│                                             │
│       Selected Works Carousel               │  ← Primary content
│                                             │
│       Email  |  Instagram                   │  ← Footer
└─────────────────────────────────────────────┘

Overlays on Page 2:
  - About modal
  - Contact modal
```

---

## 7. Component Specifications

### 7.1 Loading Screen (Page 1 only)

**Behavior:** Renders on top of all content (`z-index: 9999`) immediately on Page 1 load. Automatically fades out after 1.5 seconds. Non-interactive (`pointer-events: none` throughout).

**Visual:** Five vertical bars centered on screen, each animating in a ripple wave pattern (staggered `scaleY` keyframe, 0.8s duration, `ease-in-out`, infinite). The bars use `--primary-text` color.

**Animation:** CSS `fadeOut` keyframe (`opacity: 0`, `visibility: hidden`) applied to the `.loader` element with a `1.5s` delay and `forwards` fill mode.

**Bar spec:** `4px` wide, `40px` tall each. Animation delays staggered by `0.1s` increments (0s → 0.4s across five bars).

---

### 7.2 Page 1 — Landing

**Layout:** Single fullscreen page (`min-height: 100vh`). No navigation. No scroll. No decorative elements. The only visible content is the centered `<h1>`.

**Content:** A single `<h1>` reading "Mixed by Tristan Jung".

**Typography:**
- Font: Mansalva Regular, `font-weight: 400`
- Size: `clamp(2.5rem, 8vw, 5rem)` — fluid, scales with viewport
- Letter-spacing: `-2px`
- Color: `--primary-text`

**Entrance animation:** `fadeInUp` keyframe (opacity 0→1, translateY 30px→0, 1s, `ease-out`). Plays after the loading screen dismisses.

**Interactivity:** The `<h1>` is hoverable (`scale(1.05)`, `cursor: pointer`) and clickable. Clicking triggers the page transition to Page 2.

**Page transition on click:** Page 2 (`portfolio.html`) slides up over Page 1. Implementation approach: on click, Page 2 is loaded and positioned off-screen below (`translateY(100vh)`), then animates to `translateY(0)` while Page 1 remains fixed in place underneath. Transition duration: `~600ms`, `cubic-bezier(0.4, 0, 0.2, 1)` easing.

> **Back button behavior:** Because the transition uses JavaScript to inject `portfolio.html` as a DOM overlay (rather than true browser navigation), the browser history is not automatically updated. To support the back button returning the user to Page 1, the implementation must push a history entry on transition: `history.pushState({}, '', 'portfolio.html')`. The browser back button will then trigger a `popstate` event, which the JS should handle by reversing the transition animation (Page 2 slides back down, Page 1 is revealed).

---

### 7.3 Page 2 — Fixed Navigation

**Layout:** Full-width fixed bar at the top (`position: fixed`, `z-index: 100`). Two child containers: `.nav-left` (holds "About" link) and `.nav-right` (holds "Contact" link), using flexbox with `justify-content: space-between`.

**Padding:** `40px 64px` on desktop; `24px 24px` on mobile (`≤768px`).

**Entrance animation:** Fades in and slides down on page load via `fadeInDown` keyframe (0.8s, `ease-out`).

**Nav links:**
- Font: Mansalva Regular, `2.376rem` on desktop / `1.62rem` on mobile
- Color: `--primary-text`
- Border radius: `35px`
- Hover: `scale(1.05)` with `cubic-bezier(0.4, 0, 0.2, 1)` easing
- Backdrop filter: `blur(10px)`
- Both are anchor tags whose default behavior is intercepted by JavaScript to open their respective modals

---

### 7.4 About Modal

**Trigger:** Clicking the "About" nav link on Page 2.

**Content:** The modal contains a single prose block — no separate heading sections required. Copy is as follows:

> Tristan Jung is a mix engineer based in Jackson Hole, Wyoming. He approaches each project with care, a fresh perspective, an open mind, and open ears. His focus is to deliver your vision for your music in the best way possible, communicate, and above all else: make the music feel right to everyone involved.

The developer should render this as a single `<p>` element inside the modal body. No subheadings, no sections — just the paragraph, styled in Mansalva Regular at `1rem`, `letter-spacing: 0.02em`, color `--primary-text`, with comfortable line-height (`1.7` recommended) for readability.

**Layout:** Centered overlay with `rgba(0,0,0,0.8)` backdrop. Modal card: `max-width: 800px`, `width: 90%` (mobile: `95%`), `max-height: 90vh`, `overflow-y: auto`. Background: `#8A9BA8`. Border-radius: `30px`. Padding: `64px` desktop / `40px` mobile.

**Scroll behavior:** The backdrop itself scrolls with the page — it is not `position: fixed`. The modal card scrolls internally (`overflow-y: auto`) when its content exceeds `max-height: 90vh`. The page body scroll should be disabled (`overflow: hidden` on `<body>`) while any modal is open, so only the modal card itself is scrollable.

**Close behaviors (all three required):**
1. Clicking the × button (top-right of modal card, rotates 90° on hover)
2. Clicking the backdrop outside the modal card
3. Pressing the `Escape` key

**Entrance animation:** Modal card slides up (`translateY(50px)` → `0`) with simultaneous fade-in, 0.3s duration.

---

### 7.5 Contact Modal

**Trigger:** Clicking the "Contact" nav link on Page 2.

**Content:**
- Heading: "Get In Touch"
- Email address: `tristanjung.engineer@gmail.com`, rendered as a `mailto:` link, displayed at `1.65rem`, `font-weight: 400`
- Instagram icon: same inline SVG as the footer (see §7.7), linking to `https://www.instagram.com/1tristanjung/`, `target="_blank"`, `aria-label="Instagram"`

**Copy to clipboard — email:** Clicking the email address copies `tristanjung.engineer@gmail.com` to the clipboard via the `navigator.clipboard.writeText()` API. The default `mailto:` behavior is intercepted with `event.preventDefault()` — clicking copies to clipboard only and does not open a mail client. On successful copy, a small tooltip reading "Copied!" appears above the email address, fades in instantly, holds for `1.5s`, then fades out. Tooltip styles: Mansalva Regular, `0.75rem`, `--primary-text` color, `--bg-primary` background, `6px` border-radius, `8px` padding (`4px 8px`), no border. Positioned absolutely above the email element (`bottom: 100%`, `left: 50%`, `transform: translateX(-50%)`), `pointer-events: none`.

**Layout and behavior:** Identical to About modal (same container styles, same three close methods, same entrance animation, same scroll behavior — backdrop scrolls with page, modal card scrolls internally, body scroll disabled while open).

---

### 7.6 Selected Works Section

**Entrance animation:** Section starts at `opacity: 0`, `translateY: 30px` and transitions in when scrolled into view via `IntersectionObserver` (threshold `0.1`, rootMargin `0px 0px -50px 0px`). Transition: `0.8s ease-out`.

#### Carousel Layout

The works are displayed in a **stage-style carousel** — a single horizontal row where 5 cards are visible simultaneously, scaled by their distance from the center. The carousel is horizontally and vertically centered on the page.

| Position | Scale | Blur |
|---|---|---|
| Center (active) | 100% | none |
| Adjacent (±1) | 65% | `blur(2px)` |
| Outermost (±2) | 40% | `blur(4px)` |

**Initial state:** The carousel loads with **New Chanel** (card 1) centered. This is expected to be updated regularly after launch — the developer should make the initial index easy to change via a single variable at the top of the carousel JS module (e.g. `const INITIAL_INDEX = 0`). Cards are absolutely or flexibly positioned so all five are visible at once within the viewport, with the center card dominant. Overflow on cards beyond ±2 positions is hidden.

**Transition behavior:** When the user advances the carousel, all visible cards animate simultaneously to their new scale, blur, and position. No snapping — motion is smooth and continuous. Recommended transition: `0.4s cubic-bezier(0.4, 0, 0.2, 1)` applied to `transform` and `filter`.

#### Navigation

Three methods of navigation are supported simultaneously. There are no arrow buttons.

**Drag / swipe:**
- Desktop: click-and-drag horizontally to advance
- Mobile: touch swipe left/right
- A swipe/drag threshold should be set (e.g. `>50px` horizontal movement) to distinguish intentional navigation from accidental touches
- Direction: dragging left advances forward, dragging right goes back

**Keyboard:**
- Left (`←`) and right (`→`) arrow keys advance the carousel by one card
- The carousel must be focusable (or document-level `keydown` listener is acceptable)
- At the first or last card, the respective key does nothing

**Wrapping:** The carousel does not wrap. At the first card, leftward navigation (drag, swipe, keyboard left) does nothing. At the last card, rightward navigation does nothing. The progress dot sitting at either extreme provides implicit visual confirmation of the boundary.

#### Progress Indicator

A minimal scrubber sits below the carousel, centered, spanning a fixed width (recommended: `200px` desktop / `140px` mobile).

**Track:** A thin horizontal bar (`height: 2px`, `border-radius: 1px`). The track itself is invisible — `background: transparent` — serving only as a positional container for the dot.

**Dot:** A single circular dot (`width: 6px`, `height: 6px`, `border-radius: 50%`) in `--primary-text` color. It sits centered on the track and slides horizontally to reflect the current carousel position. At card 1 of 17, the dot sits at the left end of the track; at card 17 of 17, it sits at the right end.

**Position formula:** `left = (currentIndex / (totalCards - 1)) * trackWidth - dotRadius`

**Animation:** The dot translates smoothly with the same `0.4s cubic-bezier(0.4, 0, 0.2, 1)` timing as the carousel cards.

**Spacing:** `32px` gap between the bottom of the carousel cards and the progress indicator.

---

#### Work Card Anatomy

Each card contains two child elements:

**`.work-image`**
- Square aspect ratio recommended (cover art is typically square)
- `overflow: hidden`, `border-radius: 20px`
- Contains a cover art `<img>` (`object-fit: cover`, fills container)
- Non-center cards receive `filter: blur()` per the scale table above

**`.work-info`**
- Padding: `24px 0` (top/bottom only)
- Text aligned center
- Visible on all cards regardless of position, but naturally de-emphasized on scaled-down cards
- `.work-title`: `1.3rem`, `letter-spacing: -0.5px`, color `--primary-text`
- `.work-artist`: default size, `letter-spacing: 0.02em`, color `--primary-text`, `opacity: 0.8`

#### Click & Audio Interaction

Card click behavior depends on the card's current position:

- **Clicking a non-center card (±1 or ±2):** Advances the carousel to bring that card to center. No audio plays on this first click.
- **Clicking the center card:** Triggers audio playback (if the card has a `data-audio` attribute) or opens the Spotify link (if the card is Spotify-linked).

**Audio playback rules:**
- Clicking the center card plays its associated MP3 (`new Audio(path)`)
- Clicking the center card again while playing: pauses; clicking again: resumes
- Advancing the carousel while audio is playing: stops the current audio
- Audio cleans up (`currentAudio = null`) when the track ends naturally
- `preload: 'none'` on all audio objects for performance
- **Visual feedback:** When the center card is actively playing, a visual indicator must be shown (e.g. animated waveform icon, play/pause overlay, or highlighted border). Exact treatment at developer's discretion but must be present.

**Global state:** Two module-level variables — `currentAudio` (Audio object) and `currentCardIndex` (integer) — manage playback and carousel position.

#### Spotify-Linked Cards

Cards without a `data-audio` attribute but with a Spotify URL open that URL in a new tab when clicked as the center card. Cards may be audio-enabled or Spotify-linked, but not both.

#### Work Credits

The site includes the following 17 works:

| Title | Artist(s) | Audio (MP3) | Spotify |
|---|---|---|---|
| New Chanel | Supreme Dae ft. d4mon & LOM MAZI | ✓ | — |
| ActingUp | d4mon | — | ✓ |
| 2s!des (2s!des & Letter2Myself) | d4mon | ✓ | — |
| Work it Out | d4mon | ✓ | — |
| Flawed | d4mon, SupremeDae, LOM MAZI | ✓ | — |
| Escape the Loop | Roy Michael ft. TheMIND | ✓ | — |
| Janym (Full Album) | Jass. | — | ✓ |
| Adrenaline | VERAXON | ✓ | — |
| Wimbledon 25' | SupremeDae | ✓ | — |
| DND | LOM MAZI & SupremeDae ft. d4mon | ✓ | — |
| Far Not Far Really | LOM MAZI ft. Seddy Hendrinx | ✓ | — |
| LTAH | UoT | ✓ | — |
| 1,000 HP TO THA WHEELS | UoT | ✓ | — |
| PLAY BOUT YOU | UoT | ✓ | — |
| UoT ON MY SKIN | UoT | ✓ | — |
| Red Light Green Light | Titus Irons | ✓ | — |
| Go! | Titus Irons | ✓ | — |

---

### 7.7 Contact Footer (Page 2)

**Layout:** Centered column with `24px` gap between items.

**Items:**
1. Email address — `tristanjung.engineer@gmail.com` as a `mailto:` anchor (or Cloudflare-obfuscated equivalent)
2. Instagram icon — inline SVG linking to `https://www.instagram.com/1tristanjung/`, `target="_blank"`, `aria-label="Instagram"`

**Copy to clipboard — email:** Identical behavior to the Contact modal (see §7.5). Clicking the email address intercepts the `mailto:` default, copies the address to clipboard, and shows the same "Copied!" tooltip. The tooltip implementation should be a shared utility function used by both the modal and footer instances.

**Instagram SVG:** Custom inline SVG, `24×24px`. Rounded rectangle (stroke only) + circle (stroke only, lens) + filled dot (viewfinder).

**Hover:** Each item lifts `translateY(-3px)` on hover (`transition: 0.3s`).

**Entrance animation:** Same `IntersectionObserver` scroll-in as the Selected Works section.

---

## 8. Animation & Interaction Summary

| Element | Page | Trigger | Animation |
|---|---|---|---|
| Loading screen | 1 | Page load | Waveform bars loop; screen fades out at 1.5s |
| Hero `<h1>` | 1 | Page load | `fadeInUp` — rises in from below |
| Hero `<h1>` | 1 | Hover | `scale(1.05)` |
| Hero `<h1>` | 1 | Click | Page 2 slides up over Page 1 (~600ms) |
| Navigation bar | 2 | Page load | `fadeInDown` — slides in from top |
| Nav links | 2 | Hover | `scale(1.05)` |
| About / Contact modals | 2 | Nav click | Backdrop fade-in + card `slideUp` (0.3s) |
| Modal close `×` button | 2 | Hover | `rotate(90deg)` |
| Carousel advance | 2 | Drag, swipe, keyboard arrow key, or click non-center card | All cards simultaneously scale + blur + translate (0.4s) |
| Progress dot | 2 | Carousel advance | Slides along invisible track (0.4s, same easing as carousel) |
| Non-center card | 2 | Click | Carousel advances to bring that card to center |
| Center card (audio) | 2 | Click | Play/pause MP3 + visual playing indicator |
| Center card (Spotify) | 2 | Click | Open Spotify in new tab |
| Carousel advance while playing | 2 | Drag, swipe, or keyboard | Stops current audio |
| Selected Works section | 2 | Scroll into view | `IntersectionObserver` fade + rise (0.8s) |
| Contact footer | 2 | Scroll into view | `IntersectionObserver` fade + rise (0.8s) |
| Contact footer items | 2 | Hover | `translateY(-3px)` |
| Email address (modal + footer) | 2 | Click | Copies to clipboard; "Copied!" tooltip fades in, holds 1.5s, fades out |

All transitions use `cubic-bezier(0.4, 0, 0.2, 1)` where custom easing is specified. Default transitions fall back to `ease-out`.

---

## 9. Responsiveness

One breakpoint at `768px`:

| Property | Desktop | Mobile (≤768px) |
|---|---|---|
| Nav padding | `40px 64px` | `24px 24px` |
| Nav link font size | `2.376rem` | `1.62rem` |
| Hero `<h1>` | `clamp(2.5rem, 8vw, 5rem)` | Same (clamp handles it) |
| Section heading font size | `clamp(2.5rem, 5vw, 4rem)` | `2rem` fixed |
| Section heading margin-bottom | `64px` | `40px` |
| Carousel visible cards | 5 (center + ±1 + ±2) | 3 (center + ±1 only; ±2 hidden) |
| Carousel navigation | Drag + keyboard + click non-center | Swipe + keyboard + click non-center |
| Modal width | `90%`, max `800px` | `95%`, same max |
| Modal padding | `64px` | `40px` |
| Modal `<h2>` font size | `3rem` | `2rem` |

---

## 10. File & Asset Structure

```
/
├── index.html                        ← Page 1: Landing
├── portfolio.html                    ← Page 2: Portfolio
└── Media/
    ├── Texture/
    │   └── texture.png                   ← Seamless woven fabric tile (client to supply)
    ├── Fonts/
    │   └── Mansalva-Regular.ttf          ← Primary typeface (client supplied)
    ├── Audio/
    │   ├── new-chanel.mp3
    │   ├── 2s!des.mp3
    │   ├── work-it-out.mp3
    │   ├── flawed.mp3
    │   ├── escape-the-loop.mp3
    │   ├── adrenaline.mp3
    │   ├── wimbledon.mp3
    │   ├── dnd.mp3
    │   ├── far-not-far-really.mp3
    │   ├── ltah.mp3
    │   ├── 1000-hp-to-tha-wheels.mp3
    │   ├── play-bout-you.mp3
    │   ├── uot-on-my-skin.mp3
    │   ├── red-light-green-light.mp3
    │   └── go.mp3
    └── Photo Media/
        ├── new chanel cover.jpg
        ├── actingup cover.jpg
        ├── 2s!des cover.jpg
        ├── work it out cover.jpg
        ├── flawed cover.jpg
        ├── escapetheloop cover.jpg
        ├── janym cover.jpg
        ├── adrenaline cover.jpg
        ├── wimbledon cover.jpg
        ├── DND cover.jpg
        ├── far not far cover.jpg
        ├── ltah cover.jpg
        ├── 1000hp.jpg                    ← Shared cover art for 1,000 HP TO THA WHEELS, PLAY BOUT YOU, and UoT ON MY SKIN
        └── luv3-tape-3.jpg               ← Shared cover art for Red Light Green Light and Go!
```

**Audio asset note:** Source WAV files should be converted to MP3 for web delivery before handoff. Recommended settings: 320kbps CBR, stereo. Keep original WAVs as source files outside the project directory.

**Photo filenames contain spaces.** These should either be preserved exactly (with `src` attributes matching) or normalized to kebab-case with `src` attributes updated accordingly. Consistency is more important than the choice itself.

---

## 11. SEO & Meta

| Tag | Page 1 Value | Page 2 Value |
|---|---|---|
| `<title>` | Mixed by Tristan Jung | *(none)* |
| `charset` | UTF-8 | UTF-8 |
| `viewport` | `width=device-width, initial-scale=1.0` | `width=device-width, initial-scale=1.0` |
| `lang` | `en` | `en` |

Recommended additions for production: `<meta name="description">`, Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`), Twitter card tags, a canonical `<link>` tag, and a favicon. The production domain is `tristanjung.net` — use this for all absolute URLs in Open Graph and canonical tags.

**Keyword SEO is a priority.** The site should be optimized for search terms frequently used by the target audience — artists, labels, and music industry professionals seeking a mix engineer. The developer should research and incorporate high-value keywords into all appropriate meta tags (`<title>`, `<meta name="description">`, Open Graph fields) and any visible on-page copy where natural. Example keyword categories to target:

- Service-based: "mix engineer", "professional mixing", "music mixing services", "hire a mix engineer"
- Location-based: "mix engineer Jackson Hole", "Wyoming mix engineer"
- Genre/audience-based: terms relevant to the genres represented in Tristan's credits (hip-hop, R&B)
- Artist-adjacent: Tristan's name and the names of notable artists in his credits, as these may be searched by industry professionals verifying credits

The `<meta name="description">` on each page should be written with both search visibility and human readability in mind — it serves as the site's first impression in search results.

---

## 12. Open Items (Client to Resolve Before Launch)

1. **Visual audio indicator design.** The requirement for a playing state indicator on audio cards is confirmed, but the exact visual treatment is at the developer's discretion. Client should review and approve before launch.

2. **Favicon.** No favicon exists. Client should provide a logo mark or wordmark asset; developer will generate favicon files from it.

3. **Background texture asset.** Client to supply a seamless, tileable PNG of a woven fabric/canvas texture (approx. 200×200px). Required for both pages. Place at `Media/Texture/texture.png`.

---

## 13. Out of Scope

- Backend or server-side logic of any kind
- Contact form (email is exposed directly, no form submission)
- CMS or admin panel
- Authentication
- Analytics
- Dark/light mode toggle
- Video content
- Mastering or production services pages (mixing only, per current scope)

---

*End of PRD — Version 4.6*
