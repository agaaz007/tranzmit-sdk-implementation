# Tranzmit Exit Button SDK

AI-powered cancel button that intercepts subscription cancellations, conducts real-time voice/text exit interviews, and generates personalized win-back offers. Drop-in integration — one script tag, no backend work required.

---

## Quick Start

Add this before `</body>` on any page with a cancel button:

```html
<button id="cancel-btn">Cancel Subscription</button>

<script
  src="https://tranzmit-button-sdk-react-app.vercel.app/embed.js"
  data-api-key="eb_live_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://tranzmit-button-sdk-react-app.vercel.app"
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
  src="https://tranzmit-button-sdk-react-app.vercel.app/embed.js"
  data-api-key="eb_live_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://tranzmit-button-sdk-react-app.vercel.app"
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
  backendUrl: 'https://tranzmit-button-sdk-react-app.vercel.app',
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

## Android SDK

Native Android wrapper for the Exit Button. Same AI exit interview, same voice/text chat, same offers — runs inside your Android app with a Kotlin-native API. The modal UI, flow, and design are identical to the web version.

### Requirements

- Android API 24+ (Android 7.0)
- `RECORD_AUDIO` permission (for voice interviews)
- `INTERNET` permission

### Setup

**Step 1.** Add the dependency to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.tranzmit:exitbutton-android:1.0.0")
}
```

**Step 2.** Add permissions to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```

### Basic Integration

Show the cancel flow when the user taps your "Cancel Subscription" button:

```kotlin
import com.tranzmit.exitbutton.*

cancelButton.setOnClickListener {
    val config = ExitButtonConfig(
        apiKey = "eb_live_your_key_here",
        backendUrl = "https://tranzmit-button-sdk-react-app.vercel.app",
        userId = currentUser.id,
    )

    ExitButtonDialogFragment.newInstance(
        config = config,
        onResult = { result ->
            when (result) {
                is ExitButtonResult.Retained -> {
                    // User accepted an offer — navigate to dashboard
                    navigateTo("/dashboard")
                }
                is ExitButtonResult.Churned -> {
                    // User confirmed cancellation — process it
                    cancelSubscription()
                    navigateTo("/cancel-confirmed")
                }
                is ExitButtonResult.Error -> {
                    Log.e("ExitButton", "${result.code}: ${result.message}")
                }
                ExitButtonResult.Dismissed -> {
                    // User closed the modal without completing — do nothing
                }
            }
        }
    ).show(supportFragmentManager, "exit-button")
}
```

When the user taps cancel:

1. Full-screen modal opens with voice/text chat options
2. AI conducts a personalized exit interview (~2 min)
3. **Retained** — `onResult` fires with `ExitButtonResult.Retained`, you navigate to your dashboard
4. **Churned** — `onResult` fires with `ExitButtonResult.Churned`, you process the cancellation

### Configuration Reference

#### Required

| Parameter | Type | Description |
|-----------|------|-------------|
| `apiKey` | `String` | Your API key (`eb_live_` or `eb_test_` prefix) |

#### Business Context

| Parameter | Type | Description |
|-----------|------|-------------|
| `userId` | `String?` | User identifier. **Recommended** — auto-detection is not available on Android like it is on web. Pass your Amplitude/Mixpanel/internal user ID. |
| `planName` | `String?` | User's current plan name (e.g. `"Pro"`) |
| `mrr` | `Double?` | Monthly recurring revenue in dollars (e.g. `49.0`) |
| `accountAge` | `String?` | How long the user has been a customer (e.g. `"8 months"`) |
| `metadata` | `Map<String, Any>?` | Custom key-value pairs passed to the AI agent |

#### SDK Settings

| Parameter | Type | Description |
|-----------|------|-------------|
| `backendUrl` | `String?` | API base URL (default: `https://api.tranzmitai.com/v1`) |
| `sdkVersion` | `String?` | `"v1"` (legacy offer cards) or `"v2"` (retention decision UI) |
| `interventionAgentId` | `String?` | ElevenLabs voice agent ID (provided by Tranzmit) |
| `chatAgentId` | `String?` | ElevenLabs text chat agent ID (provided by Tranzmit) |
| `retentionApiUrl` | `String?` | v2 retention decision API URL |
| `outcomeApiUrl` | `String?` | URL for recording retention outcomes |
| `locale` | `String?` | Locale for i18n |

### Handling Outcomes (Retain & Churn Navigation)

On Android, you handle retained/churned navigation in the `onResult` callback instead of using redirect URLs like the web SDK. This gives you full control over what happens after the interview:

```kotlin
onResult = { result ->
    when (result) {
        is ExitButtonResult.Retained -> {
            // Equivalent to web's data-redirect-url="/dashboard"
            // Access session data if needed:
            val session = result.session
            Log.d("ExitButton", "User retained! Session: ${session?.id}")

            // Navigate to your dashboard/home screen
            startActivity(Intent(this, DashboardActivity::class.java))
        }
        is ExitButtonResult.Churned -> {
            // Equivalent to web's data-churn-redirect-url="/cancel-confirmed"
            // Call your cancellation API, then navigate:
            lifecycleScope.launch {
                api.cancelSubscription(userId)
                startActivity(Intent(this@MainActivity, CancelConfirmedActivity::class.java))
            }
        }
        is ExitButtonResult.Error -> {
            // Show error to user or log it
            Toast.makeText(this, "Something went wrong: ${result.message}", Toast.LENGTH_SHORT).show()
        }
        ExitButtonResult.Dismissed -> {
            // User tapped back or closed the modal — no action needed
        }
    }
}
```

#### Session Data

The `Retained` and `Churned` results include an optional `ExitButtonSession` with:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `String` | Unique session identifier |
| `userId` | `String?` | User identifier |
| `status` | `String` | `"retained"` or `"churned"` |
| `churnRiskScore` | `Int` | Risk score (0-100) |
| `createdAt` | `String` | ISO timestamp |
| `updatedAt` | `String` | ISO timestamp |
| `completedAt` | `String?` | ISO timestamp |

### Tracking Modal State

Monitor the modal's state transitions for analytics or UI updates:

```kotlin
ExitButtonDialogFragment.newInstance(
    config = config,
    onResult = { /* ... */ },
    onStateChange = { state ->
        // state is one of: "connecting", "permission", "interview",
        //                   "offers", "completing", "done", "error"
        analytics.track("exit_modal_state", mapOf("state" to state))
    }
)
```

**State flow:** `connecting` → `permission` → `interview` → `offers` → `completing` → `done`

### Theme Customization

Match the modal to your brand. Pass a `theme` map with the same properties as the web SDK:

```kotlin
val config = ExitButtonConfig(
    apiKey = "eb_live_your_key_here",
    backendUrl = "https://tranzmit-button-sdk-react-app.vercel.app",
    userId = currentUser.id,
    planName = "Pro",
    mrr = 49.0,
    accountAge = "8 months",
    theme = mapOf(
        "primaryColor" to "#6C5CE7",
        "primaryHoverColor" to "#5A4BD1",
        "backgroundColor" to "#ffffff",
        "surfaceColor" to "#f8f9fa",
        "headerBackgroundColor" to "transparent",
        "headerTextColor" to "#1a202c",
        "textColor" to "#1a202c",
        "textSecondaryColor" to "#6b7280",
        "successColor" to "#10B981",
        "errorColor" to "#EF4444",
        "borderRadius" to "12px",
        "fontFamily" to "Inter, sans-serif",
        "logoUrl" to "https://yourapp.com/logo.png",
        "logoAlt" to "Your App",
        "logoHeight" to "32px",
        "brandName" to "Acme Inc",
        "voiceIncentiveText" to "1 week free",
        "textIncentiveText" to "50% discount for 1 month",
        "chatStyle" to "bubble",
        "inputStyle" to "integrated",
        "buttonVariant" to "solid",
    ),
)
```

**All theme properties** are the same as the web SDK: `primaryColor`, `primaryHoverColor`, `backgroundColor`, `surfaceColor`, `headerBackgroundColor`, `headerTextColor`, `titleColor`, `textColor`, `textSecondaryColor`, `successColor`, `errorColor`, `borderRadius`, `fontFamily`, `logoUrl`, `logoAlt`, `logoHeight`, `brandName`, `voiceIncentiveText`, `textIncentiveText`, `chatStyle` (`flat` | `bubble`), `inputStyle` (`separate` | `integrated`), `buttonVariant` (`ghost` | `solid`), `permissionDescColor`.

### Integration Options

The SDK provides three ways to present the cancel flow, depending on your app's architecture:

#### Option 1: DialogFragment (recommended)

Full-screen dialog that manages its own lifecycle. Best for most apps.

```kotlin
ExitButtonDialogFragment.newInstance(
    config = config,
    onResult = { result -> /* handle outcome */ },
    onStateChange = { state -> /* track state */ },
).show(supportFragmentManager, "exit-button")
```

#### Option 2: Activity (for `startActivityForResult`)

Standalone Activity that returns the result via `onActivityResult`. Useful if you prefer the Activity result pattern.

```kotlin
// Launch
val intent = ExitButtonActivity.createIntent(this, config)
startActivityForResult(intent, REQUEST_CODE_EXIT_BUTTON)

// Handle result
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == REQUEST_CODE_EXIT_BUTTON && resultCode == RESULT_OK) {
        when (data?.getStringExtra(ExitButtonActivity.EXTRA_OUTCOME)) {
            "retained"  -> navigateTo("/dashboard")
            "churned"   -> { cancelSubscription(); navigateTo("/cancel-confirmed") }
            "error"     -> Log.e("ExitButton", data.getStringExtra(ExitButtonActivity.EXTRA_ERROR_MESSAGE))
            "dismissed" -> { /* no action */ }
        }
    }
}
```

**Activity extras returned:**

| Extra | Type | Description |
|-------|------|-------------|
| `ExitButtonActivity.EXTRA_OUTCOME` | `String` | `"retained"`, `"churned"`, `"error"`, or `"dismissed"` |
| `ExitButtonActivity.EXTRA_SESSION_JSON` | `String?` | Session data as JSON (for retained/churned) |
| `ExitButtonActivity.EXTRA_ERROR_MESSAGE` | `String?` | Error message (for error outcome) |
| `ExitButtonActivity.EXTRA_ERROR_CODE` | `String?` | Error code (for error outcome) |

#### Option 3: Fragment (embed in your layout)

For custom layouts — embed the fragment in any container. You manage the fragment lifecycle.

```kotlin
val fragment = ExitButtonFragment.newInstance(
    config = config,
    onResult = { result -> /* handle outcome */ },
)

supportFragmentManager.beginTransaction()
    .replace(R.id.container, fragment)
    .addToBackStack(null)
    .commit()
```

### Microphone & Voice

The SDK handles microphone permissions automatically:

1. When the voice interview starts, the SDK requests microphone access
2. Android's runtime permission dialog appears if needed
3. If the user grants permission — voice interview proceeds via WebRTC
4. If the user denies permission — the SDK automatically falls back to **text-only chat**

No additional permission handling code is needed in your app. Just make sure `RECORD_AUDIO` is declared in your manifest.

**Note:** Voice interviews require HTTPS, which the SDK uses by default. The text chat fallback works in all conditions.

### Web vs Android: Key Differences

| Feature | Web SDK | Android SDK |
|---------|---------|-------------|
| Installation | `<script>` tag or npm | Gradle dependency |
| Cancel button binding | `data-attach="#cancel-btn"` | Call `show()` in your click listener |
| Retained redirect | `data-redirect-url="/dashboard"` | Handle in `onResult` callback |
| Churned redirect | `data-churn-redirect-url="/cancel-confirmed"` | Handle in `onResult` callback |
| User ID detection | Auto-detected from PostHog, Segment, etc. | Pass `userId` explicitly |
| Theme | `data-theme` JSON string | `theme = mapOf(...)` in Kotlin |
| Modal UI & design | Identical | Identical |
| Voice interview | Identical | Identical |
| Text chat fallback | Identical | Identical |
| Offers & retention | Identical | Identical |

### Android Testing

Use an `eb_test_` prefixed API key for mock mode — no real AI calls, no charges:

```kotlin
val config = ExitButtonConfig(
    apiKey = "eb_test_your_key_here",
    backendUrl = "https://tranzmit-button-sdk-react-app.vercel.app",
)
```

### Android Troubleshooting

| Issue | Fix |
|-------|-----|
| Modal doesn't appear | Check that the dependency is added and `apiKey` is set. Check Logcat for errors. |
| "Something went wrong" in modal | Verify `apiKey` and `backendUrl` are correct. |
| Microphone permission not showing | Ensure `RECORD_AUDIO` is in your `AndroidManifest.xml`. |
| Voice not working | WebRTC requires Android API 24+. Falls back to text chat on older devices. |
| Modal blank/white | Ensure `INTERNET` permission is in your manifest and device has network. |
| `onResult` not firing | Make sure you're passing `onResult` to `newInstance()`, not setting it later. |

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
  src="https://tranzmit-button-sdk-react-app.vercel.app/embed.js"
  data-api-key="eb_test_your_key_here"
  data-attach="#cancel-btn"
  data-backend-url="https://tranzmit-button-sdk-react-app.vercel.app"
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
