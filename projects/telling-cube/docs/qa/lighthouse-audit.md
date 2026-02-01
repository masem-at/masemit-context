# Lighthouse Audit - tellingCube

**Date:** 2025-11-21
**Environment:** Development (localhost:3004)
**Story:** 3.10 - SEO & Metadata Enhancement

## How to Run Lighthouse Audit

### Option 1: Chrome DevTools (Recommended for Dev)
1. Open Chrome/Edge browser
2. Navigate to http://localhost:3004
3. Open DevTools (F12)
4. Go to "Lighthouse" tab
5. Select categories: Performance, Accessibility, Best Practices, SEO
6. Click "Analyze page load"

### Option 2: CLI (For CI/CD)
```bash
# Install Lighthouse globally
npm install -g lighthouse

# Run audit on local dev server
lighthouse http://localhost:3004 --output html --output-path ./lighthouse-report.html

# Run audit on production
lighthouse https://tellingcube.vercel.app --output html --output-path ./lighthouse-report.html
```

### Option 3: PageSpeed Insights (Production Only)
Visit: https://pagespeed.web.dev/
Enter: https://tellingcube.vercel.app

## Expected Scores (Post-Story 3.10)

| Category | Target Score | Notes |
|----------|--------------|-------|
| Performance | 80+ | May vary due to AI generation latency |
| Accessibility | 90+ | WCAG 2.1 AA compliance |
| Best Practices | 90+ | No major console errors |
| SEO | 95+ | All metadata, OG tags, favicon present |

## Key Metrics to Check

### SEO (Story 3.10 Focus)
- ✅ Document has a `<title>` element
- ✅ Document has a meta description
- ✅ Page has successful HTTP status code (200)
- ✅ Links have descriptive text
- ✅ Document has a valid `rel=canonical`
- ✅ Image elements have `[alt]` attributes
- ✅ Robots.txt is valid
- ✅ Document uses legible font sizes
- ✅ Tap targets are sized appropriately

### Accessibility
- ✅ All images have alt text
- ✅ Color contrast meets WCAG AA
- ✅ Form elements have labels
- ✅ Buttons have accessible names
- ✅ ARIA attributes valid

### Best Practices
- ✅ HTTPS enabled (on production)
- ✅ No console errors
- ✅ Images have correct aspect ratios
- ✅ No deprecated APIs

## Manual Test Results

**Tester:** James
**Date:** 2025-11-21
**URL:** http://localhost:3004

### Manual Checks Passed:
- ✅ Favicon appears in browser tab
- ✅ Page title visible: "tellingCube - Realistic Business Data in Minutes"
- ✅ Meta description present in `<head>`
- ✅ Open Graph tags present for social sharing
- ✅ No console errors on homepage
- ✅ All navigation links work
- ✅ Forms have proper labels

### Known Limitations (Dev Environment):
- ⚠️ HTTPS not enabled (localhost)
- ⚠️ Performance may be slower due to dev build

## Automated Test TODO (Post-PoC)

Consider adding automated Lighthouse CI:
```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            https://tellingcube.vercel.app
            https://tellingcube.vercel.app/scenarios
          uploadArtifacts: true
          temporaryPublicStorage: true
```

## References
- [Lighthouse Documentation](https://developer.chrome.com/docs/lighthouse/overview/)
- [Web.dev Metrics](https://web.dev/metrics/)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
