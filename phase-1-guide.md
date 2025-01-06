# Phase 1: Essential Analytics Implementation Guide

Let's start with setting up the fundamental analytics and monitoring infrastructure for your website. This phase focuses on understanding user behavior and ensuring optimal performance.

## Google Analytics Setup

First, we'll implement Google Analytics 4 (GA4) tracking. Create a new utility file:

```javascript
// src/utils/analytics.js
import { useEffect } from 'react';

// Initialize GA4
export const initGA = () => {
  if (!process.env.REACT_APP_GA_TRACKING_ID) {
    console.warn('GA tracking ID not configured');
    return;
  }

  // Load GA script
  const script = document.createElement('script');
  script.src = `https://www.googletagmanager.com/gtag/js?id=${process.env.REACT_APP_GA_TRACKING_ID}`;
  script.async = true;
  document.head.appendChild(script);

  window.dataLayer = window.dataLayer || [];
  function gtag() {
    window.dataLayer.push(arguments);
  }
  gtag('js', new Date());
  gtag('config', process.env.REACT_APP_GA_TRACKING_ID);
};

// Custom hook for page views
export const usePageTracking = () => {
  useEffect(() => {
    if (process.env.REACT_APP_GA_TRACKING_ID) {
      window.gtag('event', 'page_view', {
        page_title: document.title,
        page_location: window.location.href,
        page_path: window.location.pathname,
      });
    }
  }, []);
};
```

## Performance Monitoring Setup

Next, let's implement basic performance monitoring:

```javascript
// src/utils/performance.js
export const measurePageLoad = () => {
  if ('performance' in window) {
    window.addEventListener('load', () => {
      const timing = performance.timing;
      const loadTime = timing.loadEventEnd - timing.navigationStart;
      
      if (process.env.REACT_APP_GA_TRACKING_ID) {
        window.gtag('event', 'performance', {
          event_category: 'Performance',
          event_label: 'Page Load Time',
          value: loadTime,
        });
      }
    });
  }
};

export const measureInteraction = (interactionName) => {
  const startTime = performance.now();
  
  return () => {
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    if (process.env.REACT_APP_GA_TRACKING_ID) {
      window.gtag('event', 'interaction', {
        event_category: 'Performance',
        event_label: interactionName,
        value: Math.round(duration),
      });
    }
  };
};
```

## Feature Flags System

Create a feature flags management system:

```javascript
// src/utils/featureFlags.js
const featureFlags = {
  marketIntelligence: process.env.REACT_APP_ENABLE_MARKET_INTELLIGENCE === 'true',
  liveChat: process.env.REACT_APP_ENABLE_LIVE_CHAT === 'true',
  investmentCalculator: process.env.REACT_APP_ENABLE_INVESTMENT_CALCULATOR === 'true',
};

export const isFeatureEnabled = (featureName) => {
  return featureFlags[featureName] || false;
};

export const withFeatureFlag = (WrappedComponent, featureName) => {
  return function FeatureFlaggedComponent(props) {
    if (!isFeatureEnabled(featureName)) {
      return null;
    }
    return <WrappedComponent {...props} />;
  };
};
```

## Implementation in Main App

Update your main App component to use these utilities:

```javascript
// src/App.js
import React, { useEffect } from 'react';
import { initGA, usePageTracking } from './utils/analytics';
import { measurePageLoad } from './utils/performance';
import { isFeatureEnabled } from './utils/featureFlags';

const App = () => {
  useEffect(() => {
    initGA();
    measurePageLoad();
  }, []);

  usePageTracking();

  return (
    <div className="min-h-screen bg-[#0A0F0D]">
      {/* Your existing app content */}
      
      {isFeatureEnabled('marketIntelligence') && (
        <MarketIntelligence />
      )}
      
      {isFeatureEnabled('liveChat') && (
        <LiveChat />
      )}
    </div>
  );
};

export default App;
```

## Environment Configuration

Create a `.env` file with the necessary variables:

```env
REACT_APP_GA_TRACKING_ID=your-ga4-tracking-id
REACT_APP_ENABLE_MARKET_INTELLIGENCE=true
REACT_APP_ENABLE_LIVE_CHAT=false
REACT_APP_ENABLE_INVESTMENT_CALCULATOR=true
```

## Setting Up Google Analytics

1. Go to Google Analytics (https://analytics.google.com)
2. Create a new property
3. Choose "Web" as your platform
4. Complete the setup and get your Measurement ID (starts with "G-")
5. Add the Measurement ID to your environment variables

## Event Tracking Implementation

Create a custom event tracking utility:

```javascript
// src/utils/eventTracking.js
export const trackEvent = (category, action, label = null, value = null) => {
  if (!process.env.REACT_APP_GA_TRACKING_ID) return;

  window.gtag('event', action, {
    event_category: category,
    event_label: label,
    value: value
  });
};

// Usage example for service interaction
export const trackServiceInteraction = (serviceName, interactionType) => {
  trackEvent('Service', interactionType, serviceName);
};

// Usage example for market intelligence views
export const trackMarketView = (marketName) => {
  trackEvent('Market Intelligence', 'View', marketName);
};
```

## Testing Implementation

Create a test utility to verify your analytics setup:

```javascript
// src/utils/__tests__/analytics.test.js
import { initGA } from '../analytics';

describe('Analytics Setup', () => {
  beforeEach(() => {
    // Clear any previous GA setup
    delete window.dataLayer;
    delete window.gtag;
  });

  it('should initialize GA when tracking ID is present', () => {
    process.env.REACT_APP_GA_TRACKING_ID = 'G-XXXXXXXX';
    initGA();
    expect(window.dataLayer).toBeDefined();
  });

  it('should not initialize GA when tracking ID is missing', () => {
    process.env.REACT_APP_GA_TRACKING_ID = '';
    initGA();
    expect(window.dataLayer).toBeUndefined();
  });
});
```

## Verification Steps

1. Install the Google Analytics Debugger browser extension
2. Use the Real-Time reports in Google Analytics
3. Verify that page views are being tracked
4. Test feature flags by toggling environment variables
5. Confirm performance measurements are being recorded

Would you like me to continue with Phase 2 and 3 implementation guides, including the CMS integration and market data API setup?

Let me know if you need any clarification on Phase 1 implementation or have specific questions about the upcoming phases!