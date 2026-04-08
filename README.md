# Tranzmit Exit Button SDK

AI-powered cancel button that intercepts subscription cancellations, conducts real-time voice/text exit interviews, and generates personalized win-back offers. Drop-in integration — one script tag, no backend work required.

---

## Quick Start

Add this before `</body>` on any page with a cancel button:

```html
<button id="cancel-btn">Cancel Subscription</button>

<script
  src="https://api.tranzmitai.com/embed.js"
  data-api-key="eb_live_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://api.tranzmitai.com"
  data-redirect-url="/dashboard"
  data-churn-redirect-url="/cancel-confirmed"
></script>
```

When the user clicks cancel:

1. Modal opens with voice/text chat options
2. AI conducts a personalized exit interview (~2 min)
3. **Retained** — redirects to `data-redirect-url`
4. **Churned** — redirects to `data-churn-redirect-url`

---

## Configuration Reference

### Required

| Attribute | Description |
|-----------|-------------|
| `data-api-key` | Your API key (`eb_live_` or `eb_test_` prefix) |
| `data-attach` | CSS selector for the cancel button (e.g. `#cancel-btn`) |

### Redirects

| Attribute | Description |
|-----------|-------------|
| `data-redirect-url` | Where to send retained users (default: `/`) |
| `data-churn-redirect-url` | Where to send churned users |
| `data-prefetch-url` | URL path to prefetch analytics on page load |

### Business Context (optional)

| Attribute | Description |
|-----------|-------------|
| `data-plan-name` | User's current plan name |
| `data-mrr` | Monthly recurring revenue in dollars |
| `data-account-age` | How long the user has been a customer (e.g. `"8 months"`) |
| `data-user-id` | Override auto-detected user ID (rarely needed) |

User ID is auto-detected from PostHog, Segment, Mixpanel, Amplitude, or Intercom.

### Analytics

| Attribute | Description |
|-----------|-------------|
| `data-session-analysis` | Enable session replay analysis (default: `true`) |
| `data-analytics-provider` | `'posthog'`, `'mixpanel'`, `'amplitude'`, `'segment'`, or `'custom'` |

---

## Theme Customization

Match the modal to your brand:

```html
<script
  src="https://api.tranzmitai.com/embed.js"
  data-api-key="eb_live_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://api.tranzmitai.com"
  data-theme='{
    "primaryColor": "#6C5CE7",
    "textColor": "#1a202c",
    "backgroundColor": "#ffffff",
    "fontFamily": "Inter, sans-serif",
    "borderRadius": "12px",
    "logoUrl": "/logo.png",
    "brandName": "Acme Inc"
  }'
></script>
```

**All theme properties:** `primaryColor`, `primaryHoverColor`, `backgroundColor`, `surfaceColor`, `headerBackgroundColor`, `headerTextColor`, `titleColor`, `textColor`, `textSecondaryColor`, `successColor`, `errorColor`, `borderRadius`, `fontFamily`, `logoUrl`, `logoAlt`, `logoHeight`, `brandName`, `voiceIncentiveText`, `textIncentiveText`, `chatStyle` (`flat` | `bubble`), `inputStyle` (`separate` | `integrated`), `buttonVariant` (`ghost` | `solid`).

---

## JavaScript API

For programmatic control:

```javascript
ExitButton.init({
  apiKey: 'eb_live_your_key_here',
  backendUrl: 'https://api.tranzmitai.com',
  attach: '#cancel-btn',
  planName: 'Pro',
  mrr: 49,
  accountAge: '8 months',
  redirectUrl: '/dashboard',
  churnRedirectUrl: '/cancel-confirmed',

  onRetained: function() {
    window.location.href = '/dashboard';
  },
  onChurned: function() {
    fetch('/api/cancel-subscription', { method: 'POST' })
      .then(function() { window.location.href = '/cancel-confirmed'; });
  },
  onComplete: function(session) {
    console.log('Outcome:', session.status);
  }
});
```

### Methods

| Method | Description |
|--------|-------------|
| `ExitButton.init(config)` | Initialize with config. Returns the instance. |
| `ExitButton.start()` | Open the modal programmatically. |
| `ExitButton.prefetch()` | Pre-fetch analytics data (call on hover for faster modal open). |
| `ExitButton.close()` | Close the modal. |
| `ExitButton.destroy()` | Tear down and remove all listeners. |

### Callbacks

| Callback | When it fires |
|----------|---------------|
| `onRetained()` | User was retained. Overrides `redirectUrl`. |
| `onChurned()` | User confirmed cancellation. Overrides `churnRedirectUrl`. |
| `onComplete(session)` | Session finished. `session.status` is `'retained'` or `'churned'`. |
| `onOffer(offers)` | Win-back offers generated. |
| `onStateChange(state)` | Modal state changed. |
| `onError(error)` | Something went wrong. |
| `onReady()` | SDK attached to the cancel button. |

### Modal States

`closed` → `connecting` → `permission` → `interview` → `offers` → `completing` → `done`

---

## React SDK

For React apps, use the React package for full control:

```bash
npm install @tranzmit/react
```

Provides `<ExitButtonProvider>`, `<CancelModal>`, and hooks: `useCancelFlow`, `useVoiceState`, `useTranscript`, `useOffers`.

---

## Tracking & Analytics

Use the `onComplete` callback to send session data to your own analytics:

```javascript
ExitButton.init({
  apiKey: 'eb_live_your_key_here',
  attach: '#cancel-btn',

  onComplete: function(session) {
    analytics.track('exit_interview_completed', {
      outcome: session.status,        // 'retained' or 'churned'
      offers: session.offers,
      transcript_length: session.voiceTranscript.length
    });
  }
});
```

---

## Testing

Use an `eb_test_` prefixed API key for mock mode — no real AI calls, no charges:

```html
<script
  src="https://api.tranzmitai.com/embed.js"
  data-api-key="eb_test_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://api.tranzmitai.com"
></script>
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Nothing happens on click | Verify `data-attach` matches your button's selector. Check console. |
| Modal opens but shows error | Check `data-api-key` is valid and backend URL is correct. |
| Microphone not working | HTTPS required. User can fall back to text chat. |
| Voice cuts out | SDK auto-falls back to text on voice failure. |
| Stays on "connecting" | Check API key and network connectivity. |
| CORS errors | Verify `data-backend-url` is correct. |

---

## Support

Questions or issues? Reach out at [calendly.com/tranzmitai](http://calendly.com/tranzmitai)
