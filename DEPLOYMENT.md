# MarketIn SDK - Deployment Guide

## GitHub Setup & jsDelivr CDN Hosting

Follow these steps to publish your MarketIn SDK to GitHub and make it available via jsDelivr CDN.

### 1. Create GitHub Repository

1. Go to [GitHub](https://github.com) and create a new repository
2. Name it `marketin-sdk`
3. Make it public (required for jsDelivr)
4. Don't initialize with README (we already have one)

### 2. Push to GitHub

```bash
# Add your GitHub repository as remote (replace with your username)
git remote add origin https://github.com/MarketIN-Inc/marketin-sdk.git

# Push to GitHub
git branch -M main
git push -u origin main
```

### 3. Create a Release Tag

For jsDelivr to work properly, you need to create releases:

```bash
# Create and push a version tag
git tag v1.0.1
git push origin v1.0.1
```

Or create a release through GitHub UI:
1. Go to your repository on GitHub
2. Click "Releases" → "Create a new release"
3. Tag version: `v1.0.1`
4. Release title: `MarketIn SDK v1.0.1`
5. Description: Copy from CHANGELOG.md
6. Publish release

### 4. jsDelivr CDN URLs

Once published, your SDK will be available via jsDelivr:

**Latest version (auto-updates):**
```html
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@latest/marketin-sdk.min.js"></script>
```

**Specific version (recommended for production):**
```html
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js"></script>
```

**Development version (always latest commit):**
```html
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk/marketin-sdk.min.js"></script>
```

### 5. Update Package.json

Before publishing, update the repository URLs in `package.json`:

```json
{
  "repository": {
    "type": "git",
    "url": "git+https://github.com/MarketIN-Inc/marketin-sdk.git"
  },
  "bugs": {
    "url": "https://github.com/MarketIN-Inc/marketin-sdk/issues"
  },
  "homepage": "https://github.com/MarketIN-Inc/marketin-sdk#readme"
}
```

### 6. Update README.md

Update the badges and URLs in README.md with your actual GitHub username.

### 7. Verify jsDelivr

After pushing and creating a release:

1. Wait 5-10 minutes for jsDelivr to update
2. Visit: `https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/`
3. You should see your files listed
4. Test the direct URL: `https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js`

## NPM Publishing (Optional)

To also publish to NPM:

```bash
# Login to NPM (create account at npmjs.com first)
npm login

# Publish to NPM
npm publish
```

## Usage Examples

### Basic Integration

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>
    <!-- Your content -->
    
    <!-- MarketIn SDK -->
    <script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js"></script>
    <script>
        // Initialize SDK
        MarketIn.init({
            brandId: 'your-brand-id',
            debug: false
        });
        
        // Track conversion on button click
        document.getElementById('buy-button').addEventListener('click', function() {
            MarketIn.trackConversion({
                eventType: 'purchase',
                value: 99.99,
                currency: 'USD',
                productId: 'product-123'
            });
        });
    </script>
</body>
</html>
```

### WordPress Integration

Add to your theme's `functions.php`:

```php
function add_marketin_sdk() {
    ?>
    <script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js"></script>
    <script>
        MarketIn.init({
            brandId: '<?php echo get_option('marketin_brand_id'); ?>',
            debug: false
        });
    </script>
    <?php
}
add_action('wp_footer', 'add_marketin_sdk');
```

### Shopify Integration

Add to your theme's `theme.liquid` before `</body>`:

```html
<script src="https://cdn.jsdelivr.net/gh/MarketIN-Inc/marketin-sdk@1.0.1/marketin-sdk.min.js"></script>
<script>
    MarketIn.init({
        brandId: 'your-brand-id',
        debug: false
    });
    
    {% if template contains 'product' %}
    // Track product page view
    MarketIn.trackPageView({
        productId: '{{ product.id }}'
    });
    {% endif %}
    
    {% if checkout.id %}
    // Track purchase on thank you page
    MarketIn.trackConversion({
        eventType: 'purchase',
        value: {{ checkout.total_price | money_without_currency }},
        currency: '{{ checkout.currency }}',
        conversionRef: '{{ checkout.id }}'
    });
    {% endif %}
</script>
```

## Version Management

### Creating New Releases

```bash
# Update version in package.json
# Update CHANGELOG.md

# Commit changes
git add .
git commit -m "Release v1.0.2"

# Create and push tag
git tag v1.0.2
git push origin main
git push origin v1.0.2
```

### jsDelivr Caching

- jsDelivr caches files for 7 days
- Use specific version tags for production
- Use `@latest` only for development/testing
- Cache purging: jsDelivr automatically purges cache for new releases

## Monitoring & Analytics

### Track SDK Usage

Monitor your SDK usage through:
1. jsDelivr statistics: `https://www.jsdelivr.com/package/gh/MarketIN-Inc/marketin-sdk`
2. GitHub repository insights
3. API endpoint analytics

### Performance Monitoring

- Monitor SDK load times
- Track API response times
- Monitor error rates in production

## Security Considerations

1. **API Endpoints**: Ensure your API endpoints handle CORS properly
2. **Rate Limiting**: Implement rate limiting on your API
3. **Data Validation**: Validate all incoming data server-side
4. **HTTPS**: Always use HTTPS for API endpoints
5. **Brand ID**: Keep brand IDs secure and unique

## Support & Maintenance

1. Monitor GitHub issues for bug reports
2. Keep dependencies updated
3. Test with major browser updates
4. Maintain backward compatibility
5. Update documentation regularly

---

Your MarketIn SDK is now ready for production use via jsDelivr CDN! 🚀