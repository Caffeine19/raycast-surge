# Raycast Surge Extension - AI Coding Instructions

## Project Overview

This is an **unofficial** Raycast extension for controlling Surge (a web development and proxy utility) via its HTTP API. The extension provides a UI for switching outbound modes, managing proxy policies, toggling features, and controlling system proxy settings.

## Architecture & Key Patterns

### Core Structure

- **`src/index.jsx`**: Main command entry point with centralized state management and data fetching
- **`src/api.js`**: Centralized HTTP API client for Surge's REST endpoints
- **`src/*.jsx`**: Feature-specific React components that render List.Item elements with ActionPanels

### API Integration Pattern

All Surge communication flows through `src/api.js`, which:

- Creates an axios instance with authentication (`X-Key` header and basic auth)
- Handles both HTTP/HTTPS protocols based on user preferences
- Includes SSL certificate bypass for local development
- Returns promises for async/await consumption in components

Example API usage pattern:

```javascript
const api = require("./api")
await api(xKey, port).changeOutboundMode("proxy")
```

### Component Architecture

Components follow a consistent pattern:

- Accept `xKey`, `port`, and current state as props
- Use ActionPanel.Submenu for grouped actions
- Handle success/error with `showToast()` and `popToRoot()`
- Employ visual indicators: green checkmarks for enabled, yellow exclamation for warnings

### State Management

`index.jsx` centralizes all state using React hooks:

- Fetches all data in `useEffect` on mount
- Manages loading/error states globally
- Passes current state down to child components as props

### Raycast-Specific Conventions

- Commands defined in `package.json` with "mode": "view" or "no-view"
- Preferences for user configuration (X-Key, port, host, TLS settings)
- Uses `@raycast/api` for UI components (List, ActionPanel, showToast, etc.)

## Development Workflow

### Commands

```bash
# Development with hot reload
pnpm dev
# or: ray develop

# Production build
pnpm build
# or: ray build -e dist
```

### Code Style

- **Prettier**: No semicolons, double quotes, 120 char width, 2-space tabs
- **ESLint**: TypeScript-aware with recommended rules
- **File naming**: JSX for React components, JS for utilities, TS for pure TypeScript

### Testing Integration

Test Surge integration by configuring preferences:

1. Enable Surge's HTTP API in "More → Settings → Remote Controller"
2. Set X-Key, port (default 6171), and host in Raycast preferences
3. Use `ray develop` for live testing

## Key Integration Points

### Surge HTTP API Endpoints

- `/v1/outbound` - Outbound mode (direct/proxy/rule)
- `/v1/policy_groups` - Proxy group management
- `/v1/features/*` - Feature toggles (system_proxy, enhanced_mode, etc.)
- `/v1/profiles/*` - Profile switching and reloading
- `/v1/dns/flush` - DNS cache management

### macOS vs iOS Detection

Extension detects Surge platform via API availability (`getSystemProxyStatus()` 404 indicates iOS):

```javascript
async isMacOSVersion() {
  try {
    await this.getSystemProxyStatus()
    return true
  } catch (err) {
    return err.response?.status !== 404
  }
}
```

### Error Handling

Standard pattern: catch API errors and show user-friendly toast messages with action to check X-Key/port configuration.

## File-Specific Notes

- **`toggle-proxy.ts`**: Currently empty - intended for standalone proxy toggle command
- **`ProxyPolicies.jsx`**: Handles dynamic policy group selection with nested options
- **Component files**: Each represents a distinct Surge feature accessible from the main list

When adding new features, follow the established component pattern and extend the API module with corresponding endpoints.
