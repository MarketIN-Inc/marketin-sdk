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
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.2/marketin-sdk.min.js"></script>
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

// Track a multi-item order
MarketIn.trackConversion({
    eventType: 'purchase',
    currency: 'USD',
    cartItems: [
        { productId: 'item-1', price: 49.99, quantity: 1 },
        { productId: 'item-2', price: 24.99, quantity: 2 }
    ],
    conversionRef: 'order-456'
});

// Track with discounts and taxes
MarketIn.trackConversion({
    eventType: 'purchase',
    currency: 'USD',
    cartItems: [
        { productId: 'sku-1', price: 29.99, quantity: 1 },
        { productId: 'sku-2', price: 15.50, quantity: 2 }
    ],
    metadata: {
        discounts: [{ type: 'percentage', amount: 10 }],  // 10% off
        taxes: [{ type: 'percentage', amount: 8.25 }]     // 8.25% tax
    },
    conversionRef: 'order-789'
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

Tracks initial affiliate clicks for attribution. Automatically called when URL contains affiliate parameters (`aid`, `cid`). Persists referral data in hybrid storage for cross-session attribution.

### Conversion Tracking

```javascript
MarketIn.trackConversion({
    eventType: 'purchase',
    value: 99.99,
    currency: 'USD',
    productId: 'product-123',
    conversionRef: 'order-456', // optional, for deduplication
    cartItems: [ // optional, for multi-item orders
        { productId: 'item-1', price: 49.99, quantity: 1 },
        { productId: 'item-2', price: 24.99, quantity: 2 }
    ],
    metadata: { // optional
        orderId: 'order-456',
        customerType: 'returning'
    }
})
```

When `cartItems` is provided, the total `value` is automatically calculated from the items. If `value` is also provided, `cartItems` takes precedence.

**Parameters:**
- `eventType` (required): Type of conversion event
- `value` (optional): Conversion value (ignored when `cartItems` is provided)
- `currency` (optional): Currency code (default: 'USD')
- `productId` (optional): Product identifier (required for non-subscription events unless `cartItems` provided)
- `conversionRef` (optional): Unique reference for deduplication
- `cartItems` (optional): Array of cart items for multi-item orders
- `metadata` (optional): Additional custom data, supports `discounts` and `taxes` arrays

**Cart Items Format:**
Each cart item can include:
- `productId` (required): Product identifier
- `price` (required): Item price
- `quantity` (optional): Quantity purchased (default: 1)
- `affiliateId`, `campaignId` (optional): Override attribution for this item

**Metadata Enhancements:**
When `metadata.discounts` or `metadata.taxes` are provided, the backend computes an `effective_value` by applying discounts/taxes to the cart total. Rewards are based on the effective attributed value.

```javascript
metadata: {
    discounts: [{ type: 'percentage', amount: 10 }],  // 10% off
    taxes: [{ type: 'percentage', amount: 8.25 }]     // 8.25% tax
}
```

**Behavior & Deduplication:**
- Requires `campaignId` and `affiliateId` to be configured; otherwise conversion is skipped
- Client-side deduplication prevents duplicate conversions using `conversionRef`
- When `cartItems` is provided, the SDK automatically computes the conversion value from items
- In `trackConversion`, missing `affiliateId`/`campaignId` in `cartItems` are auto-attached from stored referral data
- Sends to authenticated `log-conversion` endpoint if `token` provided, otherwise uses public proxy
- API responses expose cart items as `cart_items` (snake_case)

**Reward Amount Calculation:**
Rewards are computed in this order: `campaign.fixed_commission_amount` > `campaign.commission_percent` > `brand.commission_percent` > `0`. Monetary math uses Decimal precision on the server.

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

For subscription lifecycle events (`subscription_created`, `subscription_renewal`, `subscription_canceled`), `productId` may be omitted. Fields like `subscriptionId`, `periodNumber`, `planId`, `interval`, and `recurringAmount` are recorded in conversion metadata when provided.

### Page Data Crawling

**Event Types:**
- `purchase` - Product purchase
- `subscription_created` - New subscription
- `subscription_renewed` - Subscription renewal
- `lead` - Lead generation
- `signup` - User registration
- Custom event types supported

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

// Referral storage utilities (advanced use)
MarketIn.saveReferralParams({ affiliateId: '123', campaignId: '456', productId: '789' });
const referral = MarketIn.getReferralParams(); // { affiliateId: '123', campaignId: '456', ... }
MarketIn.clearReferralParams();
```

## Referral Storage (Hybrid Cookie + localStorage)

The SDK uses a hybrid storage approach to persist referral data (`affiliateId`, `campaignId`, `productId`) for attribution across sessions, tabs, and delayed conversions:

- **Primary Storage**: Cookies (30-day expiry, `SameSite=Lax`, `Secure` on HTTPS) — survives browser restarts and works across tabs.
- **Fallback Storage**: localStorage — provides redundancy and works in environments where cookies are restricted.
- **Sync Mechanism**: On SDK initialization, the SDK syncs cookie and localStorage to ensure consistency.

### Benefits
- **Privacy-Safe**: Uses `SameSite=Lax` cookies to prevent CSRF while allowing top-level navigation.
- **Resilient**: Falls back to localStorage if cookies are blocked (e.g., by browser settings or extensions).
- **Cross-Tab**: Cookies ensure attribution persists across multiple tabs or windows.
- **Delayed Conversions**: Referral data survives page reloads, browser restarts, and even short-term offline periods.

### Automatic Behavior
- Referral data is saved when `trackAffiliateClick` is called (e.g., from URL params on landing).
- In `trackConversion`, missing `affiliateId`/`campaignId` in `cartItems` are auto-attached from stored referral data.
- Storage is synced on `MarketIn.init()` to handle edge cases.

### Manual Control (Advanced)
If you need to manually manage referral data (e.g., for custom integrations):

```javascript
// Save custom referral params
MarketIn.saveReferralParams({ affiliateId: '123', campaignId: '456', productId: '789' });

// Get current referral data
const referral = MarketIn.getReferralParams(); // { affiliateId: '123', campaignId: '456', ... }

// Clear referral data (e.g., on logout)
MarketIn.clearReferralParams();
```

This hybrid approach ensures reliable attribution for e-commerce checkouts, even in complex user journeys involving multiple sessions or devices.

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

## CMS Theme Integration

### WordPress

Install the official **MarketIn Integration** plugin and let it handle the SDK injection and WooCommerce conversion tracking for you.

1. Upload or install the plugin, then activate it in your WordPress dashboard.
2. Navigate to **Settings → MarketIn**. You will see inputs for **Brand ID**, **API Endpoint**, and **Private API Key**.
3. Copy your **Brand ID** from the MarketIn app under **Brand → Brands**; each brand card displays **BRAND ID: &lt;number&gt;** beneath the brand name.
4. Leave the **API Endpoint** as `https://api.marketin.now/api/v1/` unless your team provided a different environment.
5. Navigate to https://app.marketin.now/brand/api-docs, click "Generate API Key" to create a new private key, then copy it.
6. Save changes, click **Test Connection**, and wait for the success message.
7. If WooCommerce is installed, use the **Sync All Products Now** button to push your catalog into MarketIn.

After the plugin is configured, it automatically injects the SDK on every page, preserves advocate parameters, and fires conversion events on the WooCommerce thank-you page.

Check DevTools → Network for `marketin.now` requests or run a test order to confirm conversions are being captured.

## Best Practices

- Initialize early so session and affiliate detection run before redirects or conversions
- Track page views on every page load; for SPAs, call `trackPageView()` on route changes
- If your site is an SPA or uses redirects that strip query params, ensure the SDK loads before URL params are removed
- Prefer server-generated `conversionRef` (order ID) for strong idempotency
- Use HTTPS in production for secure cookies
- Keep `debug: false` in production

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

## Security & Deployment

- **CORS**: Server must allow your site origin and `Content-Type: application/json`
- **Tokens**: Provide `token` to `init()` for authenticated conversion endpoint
- **Cookies**: `mi_click` uses `SameSite=Lax` and `Secure` on HTTPS
- **Rate Limiting**: Implement rate limiting on API endpoints
- **Data Validation**: Validate all data server-side

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

### 1.0.2
- Enhanced referral parameter handling for improved tracking
- Fixed conversion attribution on thank-you pages without URL params
- Added fallback to stored referral data in trackConversion
- Improved error handling for missing affiliate/campaign IDs

### 1.0.1
- Initial release
- Basic tracking functionality
- Affiliate link support
- Conversion tracking
- Page view analytics
- Multi-item cart support with `cartItems` parameter
- Metadata enhancements for discounts and taxes
- Improved conversion attribution and reward calculation

## Support

- 📧 Email: support@marketin.now
- 📖 Documentation: [https://docs.marketin.now](https://docs.marketin.now)
- 🐛 Issues: [GitHub Issues](https://github.com/MarketIN-Inc/marketin-sdk/issues)

---

Made with ❤️ by the MarketIn team