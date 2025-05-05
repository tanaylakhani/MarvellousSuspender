# The Marvellous Suspender: Manifest V2 to V3 Migration Plan

This document outlines the step-by-step process for migrating The Marvellous Suspender Chrome extension from Manifest V2 to Manifest V3. Chrome is phasing out support for Manifest V2 extensions, so this migration is necessary to ensure the extension continues to function in Chrome.

## Background

The Marvellous Suspender is based on The Great Suspender and helps reduce Chrome's memory usage by suspending tabs that are not in use. The extension is currently using Manifest V2, but Chrome is deprecating support for V2 extensions, requiring a migration to Manifest V3.

## Major Migration Challenges

1. **Background Scripts**: Replace persistent background page with non-persistent service worker
2. **Content Security Policy**: Adapt to stricter CSP in Manifest V3
3. **Web Request API**: Replace blocking web request with declarativeNetRequest API where used
4. **Remotely Hosted Code**: Ensure all code is packaged within the extension
5. **Cross-Origin Requests**: Adjust for changes in cross-origin request handling
6. **Tab Management**: Update tab handling APIs to work with Manifest V3

## Migration Plan

### 1. Create a Development Branch
- [ ] Create a new branch for the Manifest V3 migration work
- [ ] Set up a local testing environment for the extension

### 2. Update the Manifest File
- [ ] Change manifest_version from 2 to 3
- [ ] Update minimum_chrome_version to minimum compatible version (likely "88" or higher)
- [ ] Replace "background.scripts" with "background.service_worker"
- [ ] Convert "browser_action" to "action"
- [ ] Update "web_accessible_resources" to use the new format with resources and matches properties
- [ ] Update content_security_policy to use the new format with extension_pages and sandbox directives
- [ ] Change permissions as needed (remove "webRequestBlocking" if used)

### 3. Refactor the Background Scripts
- [ ] Combine background scripts into a single service worker file
- [ ] Implement functional event-based persistence model to replace persistent background page
- [ ] Add listeners for "chrome.runtime.onInstalled" and "chrome.runtime.onStartup"
- [ ] Refactor code to handle potential service worker termination and reactivation
- [ ] Update state persistence to use chrome.storage instead of in-memory variables
- [ ] Implement persistent event listeners to receive messages from content scripts

### 4. Update Tab Management Code
- [ ] Review and update all chrome.tabs API usage for compatibility
- [ ] Update tab suspension and unsuspension mechanisms
- [ ] Update tab management logic to handle new lifecycle events

### 5. Update Message Passing
- [ ] Review and update message passing between content scripts and background
- [ ] Ensure message listeners are properly registered when service worker activates
- [ ] Implement reliable message passing that works with service worker lifecycle

### 6. Handle Content Security Policy Changes
- [ ] Update inline scripts to comply with stricter CSP rules
- [ ] Move inline event handlers to separate script files
- [ ] Remove any usage of eval(), new Function(), or other CSP-restricted features

### 7. Replace Web Request API (if used)
- [ ] Identify any usage of chrome.webRequest API, especially blocking calls
- [ ] Replace with declarativeNetRequest API or alternative approaches
- [ ] Update permissions accordingly

### 8. Fix Extension Storage
- [ ] Review storage usage and ensure compatibility with service worker lifecycle
- [ ] Replace any localStorage usage with chrome.storage.local or chrome.storage.session
- [ ] Implement proper state restoration when service worker is reactivated

### 9. Update Context Menu Implementation
- [ ] Review and update context menu creation and event handling
- [ ] Ensure context menus are created properly on service worker activation

### 10. Update External Communications
- [ ] Review and update any communication with external extensions or websites
- [ ] Update fetching of remote content to use fetch() API with proper permissions

### 11. Testing
- [ ] Test basic functionality: tab suspension and unsuspension
- [ ] Test session recovery after browser restart
- [ ] Test keyboard shortcuts
- [ ] Test context menu operations
- [ ] Test whitelist/blacklist functionality
- [ ] Test with multiple windows/tabs
- [ ] Test performance and memory usage
- [ ] Test extension behavior when idle or under system stress

### 12. Bug Fixing and Optimization
- [ ] Address any issues discovered during testing
- [ ] Optimize service worker for better performance
- [ ] Minimize service worker wake-ups to reduce battery impact
- [ ] Handle edge cases when service worker is terminated

### 13. Update Extension Documentation
- [ ] Update README.md with Manifest V3 information
- [ ] Update installation instructions as needed
- [ ] Document any behavioral changes users should expect

### 14. Prepare for Release
- [ ] Update version number in manifest.json
- [ ] Update changelog
- [ ] Create a release build
- [ ] Test the production build

### 15. Publish the Updated Extension
- [ ] Submit updated extension to Chrome Web Store
- [ ] Monitor for any issues after release
- [ ] Update user community with information about the migration

## Specific Implementation Details

### Converting background.js to Service Worker

The service worker needs to be structured differently from the current background scripts:

```javascript
// Initialization code
async function initialize() {
  // Setup state, event listeners, etc.
}

// Handle the install event
chrome.runtime.onInstalled.addListener(async (details) => {
  await initialize();
  // Additional install-specific code
});

// Handle startup
chrome.runtime.onStartup.addListener(async () => {
  await initialize();
  // Additional startup-specific code
});

// Listen for messages
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // Handle message
  return true; // if async response is needed
});
```

### Data Persistence Strategy

Since service workers can be terminated at any time:

1. Critical state should be persisted in chrome.storage.local
2. Use chrome.storage.session for temporary session data
3. Implement a state restoration mechanism on service worker activation

## Resources

- [Chrome Extensions Migration Guide](https://developer.chrome.com/docs/extensions/mv3/intro/mv3-migration/)
- [Manifest V3 Overview](https://developer.chrome.com/docs/extensions/mv3/intro/)
- [Service Worker Lifetime](https://developer.chrome.com/docs/extensions/mv3/service_workers/)