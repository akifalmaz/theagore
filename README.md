Talk to eleven of history's greatest minds — no backend, no signup, no tracking. Just you, an API key, and one HTML file.

Feynman explaining your problem back to you in the simplest possible terms. Nietzsche daring you to stop asking for permission. Sun Tzu turning your indecision into a strategy board. Mevlana asking what your heart actually wants. Eleven distinct personas, each with their own voice, their own way of thinking, and their own color — living in a single, self-contained file you can open and run in seconds.

Why this exists

Most "chat with a historical figure" tools are wrappers around a generic assistant with a name slapped on top. This one isn't. Each persona has a hand-written system prompt describing how that person actually thought — their intellectual habits, their blind spots, their way of pushing back — not just their Wikipedia summary. Feynman won't hand you an answer; he'll interrogate your assumptions until you find it yourself. Marie Curie won't indulge a grand vision; she'll ask what you're doing today. That's the point.

Features
🎭 11 personas — Richard Feynman, Albert Einstein, Marie Curie, Nikola Tesla, Leonardo da Vinci, Socrates, Confucius, Sun Tzu, Friedrich Nietzsche, Ada Lovelace, and Rumi
🎨 Original hand-drawn iconography — every character has an abstract, line-art portrait built from scratch in SVG (their signature hairstyle, headwear, or facial hair), not a scraped photo. Zero copyright baggage.
🖥️ Dock-style character switcher — a macOS/Windows-taskbar-inspired sidebar with a magnify-on-hover effect and a searchable "start menu" overlay
💬 Independent conversation threads — each character remembers your conversation with them specifically, until you refresh
🌌 Animated neural-network background — a lightweight canvas particle field that shifts color with the active persona, a small nod to the "AI meets human mind" theme
🔒 Zero backend — talks directly to the Gemini API from the browser. No server to host, no database, nothing to deploy
🇹🇷 Turkish-first UI, easily adaptable to other languages by editing the persona prompts
Tech stack

Just one file. No build step, no npm install, no framework.

Vanilla HTML / CSS / JavaScript
Google Gemini API (gemini-2.5-flash by default) called directly from the client
Hand-authored SVG for every avatar
<canvas> for the background animation
Google Fonts: Space Grotesk + IBM Plex Mono
Getting started
Get a free API key at aistudio.google.com/apikey — no credit card required.
Open index.html in a text editor and paste your key here:
js
   const API_KEY = "BURAYA_GEMINI_API_ANAHTARINI_YAPISTIR";
Open the file in your browser. That's it — double-click it, or serve it with any static file server.
⚠️ A note on the API key

This is a client-side project by design — that's what makes it a single, dependency-free file. That also means your API key is visible to anyone who opens the file's source.

✅ Fine for personal, local use
❌ Do not deploy this publicly or commit your real key to a public repo

If you want to share this project as a live website for others to use, you'll need to add a thin backend that holds the key server-side and proxies requests — happy to point you in that direction if you open an issue.

Adding your own persona

Each character is defined in two places inside the file:

js
// 1. Metadata — appears in the dock, header, and search grid
{ id: "yourperson", name: "...", title: "...", color: "#RRGGBB", tagline: "..." }

// 2. System prompt — defines how they think and speak
PERSONAS.yourperson = `...`

Optionally, add a matching entry to ACCESSORIES to give them a custom line-art portrait feature (a hat, hairstyle, or beard shape) instead of the default blank silhouette.

Disclaimer

These are AI simulations inspired by publicly known biographical and intellectual traits of historical figures — not the real people, and not verified quotes. Treat every response as a creative interpretation, not a historical source.

License

MIT — do whatever you want with it. If you build something interesting on top of this, I'd love to see it.
