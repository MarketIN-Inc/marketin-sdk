# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2024-10-09

### Added
- Initial release of MarketIn SDK
- Affiliate link tracking functionality
- Page view analytics with comprehensive data collection
- Conversion tracking for purchases, subscriptions, and custom events
- Session management with persistent cookies
- UTM parameter automatic extraction
- Page data crawling with data attributes
- Debug mode for development
- Comprehensive error handling
- Privacy-compliant implementation
- jsDelivr CDN support
- NPM package support

### Features
- `MarketIn.init()` - SDK initialization
- `MarketIn.trackPageView()` - Page view tracking
- `MarketIn.trackAffiliateClick()` - Affiliate click attribution
- `MarketIn.trackConversion()` - Conversion tracking
- `MarketIn.crawlPageData()` - Automatic product data extraction
- `MarketIn.getStatus()` - SDK status and configuration
- Automatic URL parameter detection (aid, cid, pid, utm_*)
- Cookie-based session persistence
- Deduplication for conversions
- Background API requests with error handling

### Technical
- Lightweight minified version under 10KB
- Browser support: Chrome 60+, Firefox 55+, Safari 12+, Edge 79+
- No external dependencies
- IIFE pattern for browser compatibility
- Comprehensive JSDoc documentation

### Documentation
- Complete API reference
- Usage examples
- Integration guide
- Browser compatibility information
- Privacy and GDPR compliance notes

## [Unreleased]

### Planned
- TypeScript definitions
- React/Vue.js wrapper components
- Advanced analytics dashboard
- Real-time event validation
- Webhook support for conversions
- A/B testing utilities
- Enhanced product recommendation tracking