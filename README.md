# Webflow GDPR Consent

A lightweight, dependency-free consent manager for Webflow that helps you collect GDPR-compliant consent for cookies, analytics, and marketing tools. Drop-in HTML/CSS/JS snippets, fully customizable.

## Features
- **Customizable UI**: Banner or modal style (provided modal example), easy to restyle via CSS variables.
- **Granular consent**: Essential (always on), Analytics, Marketing.
- **Event hooks**: Trigger analytics/marketing only after opt-in.
- **Zero dependencies**: Pure client-side, runs anywhere Webflow runs.

## Quick Start (Webflow)
1. **Head styles**: Copy the contents of `cookie-style.html` into Webflow → Project Settings → Custom Code → Head Code.
2. **Consent modal + JS**: Add an Embed element at the end of your `<body>` and paste the contents of `cookie.html`.
3. Publish your site. The modal will show for users without a saved consent. It stores decisions in `localStorage`.

## How It Works
- Saves consent as JSON under localStorage key `cookie_consent_v1`.
- On first visit (no saved state), it opens the consent modal.
- On accept or save, it persists choices and calls `applyScripts(consentState)` to conditionally enable services.

### Stored State Example
```json
{
  "essential": true,
  "analytics": false,
  "marketing": true,
  "ts": 1723467890123
}
```

## Wiring Your Tools
Open `cookie.html` and locate the function that applies consent (`applyScripts(state)`). Insert your integrations there as follows:

- Analytics: After the user opts in, initialize your analytics library (e.g., GA4 or GTM) and trigger any consent-related events in your data layer. Ensure scripts are only loaded after consent and avoid duplicates by guarding with a unique element ID or a runtime flag.
- Marketing: Only inject marketing pixels or SDKs after marketing consent is granted. Use on-demand script injection and add a duplicate guard (e.g., check for an existing tag by ID) before appending anything to the DOM.
- Error handling: Wrap vendor initializations in small helper functions and handle failures gracefully so a third-party outage does not break the page.
- Performance: Prefer async/deferred loading and load as little as possible on first paint; inject heavier tools only when consent is present.

Recommendations:
- Lazy-load third-party scripts only after opt-in (create `loadAnalytics()` / `loadMarketingPixel()` functions that inject `<script>` tags on demand).
- For GTM, you can push a custom event and configure GTM to load tags based on consent variables.

## Reset Consent via URL Parameter
For testing or support, you can clear the saved consent by appending a parameter to the URL:

```
https://your-domain.com/?reset-consent
```

Behavior:
- On page load, the script removes the `localStorage` entry `cookie_consent_v1` when it detects `reset-consent` in the query string.
- The parameter is then removed from the address bar via `history.replaceState` (no reload), so links don’t continue resetting consent.
- After resetting, the consent modal will show again on the next interaction/page view according to your initialization logic.

## UI and Configuration
- Categories are defined in the form in `cookie.html` with checkboxes named `analytics` and `marketing`. `essential` is always enabled.
- Change the `STORAGE_KEY` if you make breaking changes to consent categories:

```js
const STORAGE_KEY = 'cookie_consent_v1';
```

- Customize styling in `cookie-style.html`. Key CSS variables:

```css
:root {
  --cc-bg: #ffffff;
  --cc-text: #000000;
  --cc-border: #e5e5e5;
  --cc-primary: #000000;
}
```

## Reopen Consent Modal (User Link)
If you want a "Cookie Settings" link in your footer, add an Embed with:

```html
<a href="#" onclick="document.getElementById('cookie-consent').classList.remove('cc-hidden'); return false;">Cookie Settings</a>
```

Alternatively, open via JS:
```js
document.getElementById('cookie-consent').classList.remove('cc-hidden');
```

## Data Layer Example
If you use a data layer (e.g., GTM), read consent like this:
```js
const consent = JSON.parse(localStorage.getItem('cookie_consent_v1') || '{}');
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  event: 'consent_state',
  consent
});
```

## Browser Support
- Works in all modern browsers that support `localStorage`.
- Gracefully degrades: if `localStorage` is unavailable, consent won’t persist between page loads.

## Notes & Disclaimer
- This snippet is a technical aid and **not legal advice**. Consult your legal counsel for your specific use case, especially for GDPR, ePrivacy, and regional requirements.
- Ensure your privacy policy reflects your tracking tools and consent logic.

## Development
- Files:
  - `cookie-style.html` – CSS to paste into the `<head>`.
  - `cookie.html` – Modal markup and JS to paste before `</body>`.
- No build step or dependencies.

## License
MIT
