# MarketIn SDK

A lightweight JavaScript SDK for tracking website activities, affiliate links, and conversions.

[![Version](https://img.shields.io/npm/v/marketin-sdk.svg)](https://npmjs.org/package/marketin-sdk)
[![jsDelivr](https://data.jsdelivr.com/v1/package/gh/MarketIN-Inc/marketin-sdk/badge)](https://www.jsdelivr.com/package/gh/MarketIN-Inc/marketin-sdk)

## Features

- 🔗 **Affiliate Link Tracking** - Track clicks and attribute conversions
- 📊 **Page View Analytics** - Comprehensive page visit tracking
- 💰 **Conversion Tracking** - Track sales, subscriptions, and custom events
- 🍪 **Session Management** - Persistent user sessions with cookies
- 🎯 **UTM Parameter Support** - Automatic UTM parameter extraction
- 🔒 **Privacy Compliant** - GDPR and privacy-friendly implementation
- ⚡ **Lightweight** - Minified version under 10KB
- 🌐 **CDN Ready** - Available via jsDelivr

## Installation

### Via CDN (Recommended)

```html
<!-- Latest version -->
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@latest/marketin-sdk.min.js"></script>

<!-- Specific version -->
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js"></script>
```

### Via NPM

```bash
npm install marketin-sdk
```

### Self-hosted

Download `marketin-sdk.min.js` and host it on your server:

```html
<script src="/path/to/marketin-sdk.min.js"></script>
```

## CMS Theme Integration

### WordPress
- **Create or select a child theme:** Avoid editing parent themes directly so updates do not overwrite the integration. If you do not already have a child theme, generate one (e.g., via `wp scaffold child-theme` or manual `style.css` + `functions.php`). Activate it in `Appearance > Themes`.
- **Host the SDK:** Use the jsDelivr CDN or self-host `marketin-sdk.min.js` inside the child theme (e.g., `wp-content/themes/your-child/js/marketin-sdk.min.js`).
- **Enqueue the script:** Add the following to the child theme `functions.php` so the SDK loads on the frontend. Replace the CDN URL if self-hosting.

```php
function marketin_enqueue_sdk() {
        if (is_admin()) {
                return;
        }

        wp_enqueue_script(
                'marketin-sdk',
                'https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@latest/marketin-sdk.min.js',
                array(),
                null,
                true
        );

        wp_add_inline_script(
                'marketin-sdk',
                'window.addEventListener("DOMContentLoaded", function() { MarketIn.init({ brandId: "YOUR_BRAND_ID", debug: false }); });'
        );
}
add_action('wp_enqueue_scripts', 'marketin_enqueue_sdk');
```

- **Localize dynamic data (optional):** Use `wp_localize_script` or `wp_add_inline_script` to pass per-page campaign data when available from WordPress.
- **Capture conversions:** Hook into commerce plugins (e.g., WooCommerce `woocommerce_thankyou` action) or form submissions and call `MarketIn.trackConversion`. Example for WooCommerce:

```php
add_action('woocommerce_thankyou', function ($order_id) {
    $order = wc_get_order($order_id);
    if (!$order) {
        return;
    }

    $payload = array(
        'eventType' => 'purchase',
        'value' => (float) $order->get_total(),
        'currency' => $order->get_currency(),
        'productId' => implode('-', wp_list_pluck($order->get_items(), 'product_id')),
        'conversionRef' => 'order-' . $order_id,
        'metadata' => array(
            'orderId' => $order_id,
            'customerEmail' => $order->get_billing_email()
        )
    );

    wp_add_inline_script('marketin-sdk', 'MarketIn.trackConversion(' . wp_json_encode($payload) . ');');
});
```

- **Subscriptions:** For plugins like WooCommerce Subscriptions or MemberPress, listen for renewal hooks and send `eventType` values such as `subscription_created` or `subscription_renewed` with the relevant plan metadata.
- **Verify:** Load a public page, open DevTools console, ensure `[MarketIn SDK] SDK initialized` appears. Confirm outbound requests to `log-activity` succeed.

### Shopify (Liquid)
- **Upload the script:** In `Online Store > Themes > Edit code`, add `marketin-sdk.min.js` to `assets/`. Alternatively use the CDN URL.
- **Include globally:** Edit `layout/theme.liquid` and insert the script before `</body>` (or use `{{ 'marketin-sdk.min.js' | asset_url | script_tag }}` when self-hosted).
- **Initialize:** Immediately after the script include, add an inline snippet:

```liquid
<script>
document.addEventListener('DOMContentLoaded', function () {
    if (window.MarketIn) {
        window.MarketIn.init({
            brandId: '{{ shop.id }}',
            debug: {{ shop.metafields.marketin.debug | default: false | json }}
        });
    }
});
</script>
```

- **Dynamic data:** Use Liquid variables (`customer.id`, `cart.items`, `page_title`) to enrich `MarketIn.trackConversion` calls in templates like `checkout.liquid` or `order-status.liquid`.
- **Capture conversions:** In `checkout.liquid` (for Shopify Plus) or the order status page scripts, invoke `MarketIn.trackConversion` with order totals and IDs:

```liquid
{% if first_time_accessed %}
    <script>
    document.addEventListener('DOMContentLoaded', function () {
        if (window.MarketIn) {
            window.MarketIn.trackConversion({
                eventType: 'purchase',
                value: {{ checkout.total_price | money_without_currency | replace: ',', '' }},
                currency: '{{ checkout.currency }}',
                productId: '{{ checkout.line_items | map: "product_id" | join: "-" }}',
                conversionRef: 'order-{{ checkout.order_number }}'
            });
        }
    });
    </script>
{% endif %}
```

- **Subscriptions:** If using apps that expose subscription data (e.g., Recharge), subscribe to their JS callbacks/webhooks and forward events via `MarketIn.trackConversion` with `eventType` set to `subscription_created`, `subscription_renewed`, etc.
- **Test:** Preview the theme, perform a cart action, and verify requests fire in the network tab.

### Webflow
- **Add the script globally:** In Webflow Designer go to `Project Settings > Custom Code > Footer Code`, paste the CDN script tag and an initialization snippet:

```html
    <script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@latest/marketin-sdk.min.js"></script>
<script>
    document.addEventListener('DOMContentLoaded', function () {
        MarketIn.init({ brandId: 'your-brand-id', debug: false });
    });
</script>
```

- **Per-page overrides:** For campaign-specific landing pages, override `brandId` or add `campaignId` in the page-level `Custom Code > Before </body>` settings.
- **Product data:** Use custom attributes (e.g., `data-marketin-product`) within Webflow elements to power `MarketIn.crawlPageData()` if you enable crawling.
- **Capture conversions:** Use Webflow’s form success callbacks or ecommerce `Order Confirmation` page custom code to call `MarketIn.trackConversion`. Example form submission snippet:

```html
<script>
    document.addEventListener('w-form-done', function (event) {
        if (window.MarketIn) {
            window.MarketIn.trackConversion({
                eventType: 'lead',
                currency: 'USD',
                value: 0,
                conversionRef: 'form-' + Date.now(),
                metadata: { formId: event.target.id }
            });
        }
    });
</script>
```

- **Subscriptions:** For Webflow Ecommerce, add code on the order confirmation page to send subscription-related events when applicable, populating `subscriptionId`, `planId`, and `recurringAmount` fields.
- **Publish and verify:** Publish the site, open the live domain, and confirm the SDK loads and logs page views.

### Other CMS Platforms
- **Generic include:** Most CMS systems (Drupal, Joomla, Ghost) let you edit theme templates or inject footer scripts. Insert the CDN `<script>` tag in the global layout and call `MarketIn.init({ brandId: 'your-brand-id' });` on DOM ready.
- **Module/plugin approach:** If the CMS supports script injection plugins/modules, configure one to load the script after page render. Prefer footer placement to avoid blocking rendering.
- **Configuration management:** Store `brandId`, `campaignId`, and `apiEndpoint` in CMS configuration screens or environment variables so non-technical users can update without code edits.
- **Capture conversions:** Identify the CMS event hooks (e.g., Drupal Commerce checkout completion, Joomla VirtueMart orders) and trigger `MarketIn.trackConversion` with order totals and references. For recurring billing modules, send `subscription_*` events with plan metadata.
- **QA:** Use staging environments to confirm that CDN delivery works and that CSP headers allow external scripts (`script-src` should include the CDN domain or use a hash/nonce).

## Laravel Integration
- **Load the SDK:** In your main Blade layout (`resources/views/layouts/app.blade.php` or similar) add the CDN script after your compiled assets. If you prefer self-hosting, expose `marketin-sdk.min.js` via Laravel Mix/Vite and reference the built asset path.

```blade
<!-- resources/views/layouts/app.blade.php -->
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    @vite(['resources/js/app.js'])
</head>
<body>
    @yield('content')

    <script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@latest/marketin-sdk.min.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', function () {
            if (window.MarketIn) {
                window.MarketIn.init({
                    brandId: @json(config('services.marketin.brand_id')),
                    campaignId: @json(session('campaign_id')),
                    debug: @json(config('app.debug'))
                });
            }
        });
    </script>
</body>
</html>
```

- **Share dynamic data:** Populate `brandId`, campaign identifiers, or tokens from `config/services.php`, environment variables, or middleware that captures affiliate parameters and stores them in the session. Use Blade `@php` blocks or `@json()` helpers to serialize PHP arrays into the inline script safely.
- **Handle conversions:** Invoke `MarketIn.trackConversion` once an order or subscription is confirmed. For example, hook into your controller after payment success:

```blade
@push('scripts')
<script>
    if (window.MarketIn) {
        window.MarketIn.trackConversion({
            eventType: 'purchase',
            value: @json($order->total),
            currency: @json($order->currency),
            productId: @json($order->lineItems->pluck('sku')->implode('-')),
            conversionRef: @json('order-' . $order->id),
            metadata: @json(['customerEmail' => $order->customer_email])
        });
    }
</script>
@endpush
```

- **Livewire and Inertia:** For Livewire components, dispatch a browser event (`$this->dispatchBrowserEvent('marketin:conversion', [...])`) and listen in JavaScript to call the SDK. With Inertia or SPA routes, re-run `MarketIn.trackPageView()` on navigation changes to ensure single-page transitions are recorded.
- **Subscriptions:** When using Laravel Cashier or Spark, subscribe to billing events (e.g., `InvoicePaymentSucceeded`) and pass subscription IDs, plan IDs, and amounts through `MarketIn.trackConversion` with `eventType` values like `subscription_created` or `subscription_renewed`.
- **Security considerations:** If you enforce Content Security Policy headers, whitelist the CDN domain or self-host the script. Confirm `script-src` allows the inline initialization snippet (`'unsafe-inline'`, hash, or nonce).

## Quick Start

```javascript
// Initialize the SDK
MarketIn.init({
    brandId: 'your-brand-id',
    debug: false // Set to true for development
});

// Track a conversion
MarketIn.trackConversion({
    eventType: 'purchase',
    value: 99.99,
    currency: 'USD',
    productId: 'product-123'
});
```

## API Reference

### Initialization

```javascript
MarketIn.init(options)
```

**Options:**
- `brandId` (required): Your brand identifier
- `campaignId` (optional): Default campaign ID
- `debug` (optional): Enable debug logging (default: false)
- `apiEndpoint` (optional): Custom API endpoint

### Page View Tracking

```javascript
MarketIn.trackPageView(options)
```

Automatically tracks page views with comprehensive data including:
- URL and referrer
- Screen resolution and language
- UTM parameters
- Session information

**Options (all optional):**
- `url`: Page URL (default: current URL)
- `referrer`: Referrer URL (default: document.referrer)
- `sessionId`: Session identifier
- `campaignId`, `affiliateId`, `brandId`: Override defaults

### Affiliate Click Tracking

```javascript
MarketIn.trackAffiliateClick({
    affiliateId: 'affiliate-123',
    campaignId: 'campaign-456',
    productId: 'product-789', // optional
    clickId: 'unique-click-id' // optional, auto-generated if not provided
})
```

Tracks initial affiliate clicks for attribution. Automatically called when URL contains affiliate parameters (`aid`, `cid`).

### Conversion Tracking

```javascript
MarketIn.trackConversion({
    eventType: 'purchase',
    value: 99.99,
    currency: 'USD',
    productId: 'product-123',
    conversionRef: 'order-456', // optional, for deduplication
    metadata: { // optional
        orderId: 'order-456',
        customerType: 'returning'
    }
})
```

**Event Types:**
- `purchase` - Product purchase
- `subscription_created` - New subscription
- `subscription_renewed` - Subscription renewal
- `lead` - Lead generation
- `signup` - User registration
- Custom event types supported

### Subscription Tracking

```javascript
MarketIn.trackConversion({
    eventType: 'subscription_created',
    value: 29.99,
    currency: 'USD',
    subscriptionId: 'sub-123',
    planId: 'plan-premium',
    interval: 'monthly',
    recurringAmount: 29.99,
    subscriptionStatus: 'active'
})
```

### Tracking Purchases and Subscriptions

MarketIn provides built-in helpers so you don’t need to wire anything manually:

```javascript
// Purchase helper
MarketIn.handleTrackConversion({
    productId: 'sku-123',
    value: 99.99,
    currency: 'NGN'
});

// Subscription helper
MarketIn.handleSubscriptionSubmit({
    eventType: 'subscription_started',
    recurringAmount: 500,
    currency: 'NGN'
});
```

Both helpers delegate to `MarketIn.trackConversion()` — make sure your integration has already established `campaignId` and `affiliateId` (via URL parameters or by passing them during `MarketIn.init`) so conversions can be attributed correctly.

### Page Data Crawling

```javascript
MarketIn.crawlPageData()
```

Automatically extracts product information from pages with `data-marketin-product` attributes:

```html
<div data-marketin-product="product-123" 
     data-marketin-product-name="Premium Widget"
     data-marketin-product-price="99.99"
     data-marketin-product-category="Electronics">
    Product content...
</div>
```

### URL Parameters

The SDK automatically recognizes these URL parameters:

**Short form (recommended):**
- `aid` - Affiliate ID
- `cid` - Campaign ID  
- `pid` - Product ID
- `mi_click` - Click ID for deduplication

**Legacy form:**
- `affiliate_id` - Affiliate ID
- `campaign_id` - Campaign ID
- `product_id` - Product ID

**UTM Parameters:**
- `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`

### Utility Methods

```javascript
// Get SDK status and configuration
const status = MarketIn.getStatus();
console.log(status.version); // "1.0.1"
console.log(status.initialized); // true/false
```

## Configuration

### Default Configuration

```javascript
{
    apiEndpoint: 'https://api.marketin.now/api/v1/',
    debug: true,
    sessionId: null, // Auto-generated
    affiliateId: null,
    campaignId: null,
    brandId: null
}
```

### Debug Mode

Enable debug mode to see detailed logging:

```javascript
MarketIn.init({
    brandId: 'your-brand-id',
    debug: true
});
```

## Browser Support

- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+
- IE 11+ (with polyfills)

## Privacy & GDPR

The SDK is designed to be privacy-compliant:
- No personally identifiable information is collected by default
- Uses first-party cookies only
- Respects Do Not Track headers
- Session data is not persistent across browser sessions

## Error Handling

The SDK includes comprehensive error handling:
- Failed API requests are logged but don't throw errors
- Invalid parameters are validated with helpful warnings
- Network failures are handled gracefully

## Development

```bash
# Clone the repository
git clone https://github.com/MarketIN-Inc/marketin-sdk.git
cd marketin-sdk

# No build process required - edit the source files directly
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

### 1.0.1
- Initial release
- Basic tracking functionality
- Affiliate link support
- Conversion tracking
- Page view analytics

## Support

- 📧 Email: support@marketin.now
- 📖 Documentation: [https://docs.marketin.now](https://docs.marketin.now)
- 🐛 Issues: [GitHub Issues](https://github.com/MarketIN-Inc/marketin-sdk/issues)

---

Made with ❤️ by the MarketIn team