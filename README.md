# 🐾 CatSwipe

A swipe-based cat discovery web app. Browse 20 freshly fetched cats, swipe right on the ones you love, left on the ones you don't — then get your cat personality type and see all your favourites at the end.

**Live demo:** `https://your-username.github.io/catswipe`

---

## Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [API Integration](#api-integration)
- [API Key Security](#api-key-security)
- [Personality Types](#personality-types)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Tech Stack](#tech-stack)
- [Deployment](#deployment)
- [Known Limitations](#known-limitations)

---

## Features

| Feature | Description |
|---|---|
| 🐱 20 random cats | Fetched fresh from the [Cataas API](https://cataas.com) on every session |
| 👆 Swipe gestures | Touch drag on mobile, mouse drag on desktop |
| ⌨️ Keyboard support | `←` / `→` arrow keys to swipe without touching the card |
| 🤖 AI descriptions | Gemini 2.0 Flash analyses each cat photo and writes a playful caption |
| 📊 Progress tracking | Live dots + progress bar showing liked / passed as you go |
| 🏆 Personality match | Your like count maps to one of 5 fixed personality types at the end |
| 🖼️ Liked grid | Summary screen shows a photo grid of every cat you liked |
| 🔁 Restart | Fetch a brand-new batch of 20 cats with one tap |
| 📱 Responsive | Works on phones, tablets, and desktops |
| 🚫 No login | Fully client-side — no accounts, no tracking, no data stored |

---

## Project Structure

```
catswipe/
├── index.html      # Main app — loading screen, card stack, summary
├── about.html      # About page — how it works, features, tech stack
├── style.css       # Shared stylesheet for all pages
└── script.js       # All JavaScript logic (if refactored out of index.html)
```

> **Note:** `script.js` is the extracted version of the inline `<script>` block in `index.html`. Both approaches work — if you use `script.js`, replace the `<script>...</script>` block in `index.html` with `<script src="script.js"></script>`.

---

## How It Works

### Loading phase

On page load, `preload()` builds 20 unique Cataas URLs with cache-busting query parameters (`?v=timestamp_index`) and preloads all images before showing the first card. A progress bar tracks loading.

```js
function buildUrls() {
  return Array.from({ length: TOTAL }, (_, i) =>
    `https://cataas.com/cat?v=${Date.now()}_${i}&width=600`
  );
}
```

### Card stack

Three cards are rendered at a time — the current card on top, two behind it slightly scaled and offset to create a depth effect. When the top card is swiped away, the cards behind animate forward.

```
┌─────────────┐   ← current card (interactive, full size)
 ┌───────────┐    ← card-back-1 (scale 0.95, offset 16px down)
  ┌─────────┐     ← card-back-2 (scale 0.90, offset 32px down)
```

### Swipe detection

The swipe system listens to both mouse and touch events. Dragging the card rotates it slightly (7% of horizontal offset). When released, if the drag exceeded 26% of the viewport width the card flies off; otherwise it snaps back.

```
Threshold = window.innerWidth × 0.26
```

### AI description

When each cat image loads, it is converted to base64 using `FileReader` and sent to the Gemini API along with a prompt. The real MIME type from the blob is captured and passed to Gemini — not hardcoded — so GIF, PNG, WebP, and JPEG cats all work correctly.

```js
const { base64, mimeType } = await urlToBase64(imgUrl);
// mimeType comes from blob.type — e.g. "image/gif", "image/png", "image/webp"
```

### Summary

After all 20 cards are swiped, the app shows:
1. Like / pass counts
2. Personality type (based on like count range)
3. Photo grid of every liked cat

---

## Getting Started

No build tools, no npm, no dependencies. Just open the files.

**Option 1 — Open directly in browser**

```bash
# Clone the repo
git clone https://github.com/your-username/catswipe.git
cd catswipe

# Open index.html in your browser
open index.html         # macOS
start index.html        # Windows
xdg-open index.html     # Linux
```

**Option 2 — Use VS Code Live Server (recommended)**

1. Install the [Live Server extension](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) in VS Code
2. Right-click `index.html` → **Open with Live Server**
3. App opens at `http://localhost:5500`

> Live Server is recommended because some browsers restrict `fetch()` requests from `file://` URLs, which would break the Cataas image loading and Gemini API calls.

---

## API Integration

### Cataas (Cat as a Service)

Cat photos are fetched from [cataas.com](https://cataas.com) — a free, open API that returns random cat images. No API key required.

```
GET https://cataas.com/cat?v={timestamp}_{index}&width=600
```

The `v` parameter is a cache-busting value combining `Date.now()` and the card index, ensuring each of the 20 cats is unique per session.

### Gemini 2.0 Flash (Google AI)

Used for generating cat descriptions. Each image is sent as base64 inline data along with a text prompt.

**Endpoint:**
```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=YOUR_KEY
```

**Request body:**
```json
{
  "contents": [{
    "parts": [
      {
        "inline_data": {
          "mime_type": "image/jpeg",
          "data": "<base64 string>"
        }
      },
      {
        "text": "Describe this cat in under 50 words..."
      }
    ]
  }],
  "generationConfig": { "maxOutputTokens": 100 }
}
```

**Response path:**
```js
data.candidates[0].content.parts[0].text
```

---

## API Key Security

The Gemini API key is currently hardcoded in `script.js` / `index.html`. This is visible to anyone who views the page source. To protect it:

### Recommended — HTTP referrer restriction (no backend needed)

1. Go to [console.cloud.google.com/apis/credentials](https://console.cloud.google.com/apis/credentials)
2. Delete the current exposed key
3. Create a new key → Edit → set restrictions:

| Setting | Value |
|---|---|
| Application restriction | HTTP referrers (websites) |
| Allowed referrers | `https://your-username.github.io/*` and `http://localhost/*` |
| API restriction | Restrict key → Generative Language API only |

4. Set a daily quota limit under **APIs & Services → Generative Language API → Quotas**
5. Replace the key in `script.js` with the new restricted key

A restricted key only works when requests come from your domain. Someone copying the key into Postman or their own code will receive a `403 Forbidden` from Google.

### Alternative — Vercel serverless proxy (key fully hidden)

Move the API call to a serverless function (`api/describe.js`) that reads the key from an environment variable. The browser never sees the key.

```
Browser → /api/describe (Vercel function) → Gemini API
```

See `api/describe.js` if included in the repo for the proxy implementation.

---

## Personality Types

After completing all 20 swipes, the summary screen displays a personality type based on how many cats you liked:

| Likes | Personality Type |
|---|---|
| 0 – 4 | 🪨 The Unmoved Stoic |
| 5 – 8 | 🧐 The Discerning Critic |
| 9 – 12 | ⚖️ The Balanced Realist |
| 13 – 16 | 🥰 The Warm-Hearted Softie |
| 17 – 20 | 👑 The Supreme Cat Monarch |

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `→` Arrow Right | Like the current cat |
| `←` Arrow Left | Pass on the current cat |

Keyboard input is disabled during loading and after all cards are swiped. A brief nudge animation plays before the card flies off so the feedback is visible.

---

## Tech Stack

| Technology | Purpose |
|---|---|
| HTML5 | Page structure |
| CSS3 | Styling, animations, responsive layout |
| Vanilla JavaScript | All app logic — no frameworks |
| [Cataas API](https://cataas.com) | Random cat images |
| [Gemini 2.0 Flash](https://ai.google.dev) | AI-generated cat descriptions |
| [Google Fonts](https://fonts.google.com) | Fraunces (display) + DM Sans (body) |
| Touch Events API | Mobile swipe detection |
| FileReader API | Image-to-base64 conversion for Gemini |
| CSS Animations | Card transitions, loading states, fade-ins |

Zero npm dependencies. Zero build step. Open the HTML file and it works.

---

## Deployment

### GitHub Pages

1. Push the project to a GitHub repository
2. Go to **Settings → Pages**
3. Set source to **Deploy from a branch** → `main` → `/ (root)`
4. Your site will be live at `https://your-username.github.io/repository-name`

> Make sure your Gemini API key has an HTTP referrer restriction set to `https://your-username.github.io/*` before going live.

### Local testing

Use VS Code Live Server or any static file server:

```bash
# Python (built-in)
python -m http.server 8000
# then open http://localhost:8000

# Node.js (if installed)
npx serve .
# then open http://localhost:3000
```

---

## Known Limitations

- **API key is client-side** — visible in page source. Mitigate with HTTP referrer restrictions and a daily quota cap (see [API Key Security](#api-key-security)).
- **Cataas rate limits** — occasionally a cat image fails to load (shows 😿). The app handles this gracefully and continues.
- **Gemini descriptions require internet** — if offline or if the API call fails, the card shows a fallback phrase: *"A wonderfully enigmatic cat."*
- **No persistence** — session results are in-memory only. Refreshing the page resets everything.
- **20 cats per session** — hardcoded as `const TOTAL = 20` in the script. Change this value to adjust.

---

## License

This project is open source. Do whatever you like with it — just don't sell the cats.

---

*Built with HTML, CSS, vanilla JavaScript, and an unreasonable love of cats. 🐾*
