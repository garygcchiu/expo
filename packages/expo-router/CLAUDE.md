# Expo Router

File-based routing library for React Native and web applications. Built on top of React Navigation, it provides automatic route generation from file structure, deep linking, typed routes, and cross-platform navigation.

// TODO(@ubax): verify correctness
## Structure

```
├── src/
│   ├── index.tsx              # Main entry point
│   ├── exports.ts             # Public API exports
│   ├── ExpoRoot.tsx           # Root component wrapper
│   ├── Route.tsx              # Route node definitions and context
│   ├── hooks.ts               # Navigation hooks (useRouter, usePathname, etc.)
│   ├── imperative-api.tsx     # router object for imperative navigation
│   ├── types.ts               # TypeScript type definitions
│   │
│   ├── getRoutes.ts           # Metro require context → route tree conversion
│   ├── getRoutesCore.ts       # Core route parsing with specificity scoring
│   ├── getReactNavigationConfig.ts  # React Navigation config generation
│   ├── getLinkingConfig.ts    # Deep linking configuration
│   ├── matchers.tsx           # Route segment pattern matching
│   │
│   ├── global-state/          # State management
│   │   ├── router-store.tsx   # Zustand store for router state
│   │   ├── routing.ts         # Navigation queue and routing functions
│   │   ├── routeInfo.ts       # Current route information context
│   │   └── serverLocationContext.ts  # Server-side location context
│   │
│   ├── layouts/               # Navigation layouts
│   │   ├── Stack.tsx          # Native stack navigator
│   │   ├── Tabs.tsx           # Tab navigator
│   │   ├── Drawer.tsx         # Drawer navigator
│   │   └── withLayoutContext.tsx  # Layout context HOC
│   │
│   ├── native-tabs/           # Native bottom tabs (iOS UITabBar, Android BottomNav)
│   │   ├── NativeTabsView.tsx # React Native Screens integration
│   │   └── appearance.ts      # iOS appearance styling
│   │
│   ├── link/                  # Link component
│   │   ├── Link.tsx           # Main Link component with Preview/Menu/Zoom
│   │   ├── href.ts            # Href resolution logic
│   │   ├── preview/           # Link preview UI (iOS peek/pop style)
│   │   └── zoom/              # Apple-style zoom transitions
│   │
│   ├── views/                 # Built-in screens
│   │   ├── Navigator.tsx      # Custom navigator with Slot
│   │   ├── ErrorBoundary.tsx  # Error handling
│   │   ├── Sitemap.tsx        # Route introspection
│   │   └── Unmatched.tsx      # 404 screen
│   │
│   ├── fork/                  # Forked React Navigation code
│   │   ├── NavigationContainer.tsx  # Custom NavigationContainer
│   │   ├── getStateFromPath.ts      # URL → navigation state
│   │   ├── getPathFromState.ts      # Navigation state → URL
│   │   └── native-stack/            # Native stack navigator fork
│   │
│   ├── color/                 # Platform color utilities
│   ├── primitives/            # UI components (Icon, Label, Badge)
│   ├── loaders/               # Data loader support (RSC)
│   ├── rsc/                   # React Server Components
│   ├── testing-library/       # Testing utilities
│   └── __tests__/             # Jest tests
│
├── plugin/                    # Expo router config plugin
│   └── src/index.ts           # Plugin entry point
├── ios/                       # Native iOS code (Swift)
├── android/                   # Native Android code
├── entry.js                   # Module entry point
└── build/                     # Compiled JavaScript output
```

## Testing

Run tests with jest-expo multi-platform presets:

```bash
# Run all tests
yarn test

# Run specific test file
yarn test src/__tests__/navigation.test.ios.tsx
```

### Testing Patterns

Tests use the custom `renderRouter` testing utility:

```typescript
import { renderRouter, screen } from '../testing-library';
import { router } from '../imperative-api';
import { act } from '@testing-library/react-native';

it('can navigate between routes', () => {
  renderRouter({
    index: () => <Text testID="index">Index</Text>,
    'profile/[id]': () => <Text testID="profile">Profile</Text>,
  });

  expect(screen.getByTestId('index')).toBeVisible();

  act(() => router.push('/profile/123'));

  expect(screen.getByTestId('profile')).toBeVisible();
  expect(screen).toHavePathname('/profile/123');
});
```

**Key testing utilities:**
- `renderRouter(routes, options)` - Render router with mock route configuration
- `screen.getPathname()` - Get current pathname
- `screen.getSegments()` - Get route segments array
- `screen.getSearchParams()` - Get search parameters
- `testRouter.navigate/push/replace/back()` - Navigation with assertions

**Platform-specific tests:**
- `.test.ios.tsx` - iOS tests
- `.test.android.tsx` - Android tests
- `.test.web.tsx` - Web tests
- `.test.node.ts` - Node.js tests

## Key Concepts

### File-Based Routing Conventions

- `page/index.tsx` → `/page`
- `post/[id].tsx` → `/post/:id` (dynamic segment)
- `[...rest].tsx` → catch-all route
- `(group)/_layout.tsx` → layout group (invisible in URL)
- `+not-found.tsx` → 404 handling
- `+api.ts` → API route

### Route Processing Pipeline

1. Metro `require.context()` collects route files at build time
2. `getRoutes()` converts file paths to `RouteNode` tree
3. Routes scored for specificity (more specific routes win)
4. `getReactNavigationConfig()` generates React Navigation config
5. `getLinkingConfig()` creates deep linking configuration


// TODO(@ubax): verify correctness
### State Management

- **RouterStore** (`global-state/router-store.tsx`): Zustand store managing navigation state
- **Routing Queue** (`global-state/routing.ts`): Batches navigation actions to prevent race conditions
- **Route Context**: `CurrentRouteContext` provides route info at layout boundaries

### Platform-Specific Code

Use file extensions for platform variants:
- `.ios.tsx` - iOS-specific
- `.android.tsx` - Android-specific
- `.web.tsx` - Web-specific
- `.native.tsx` - iOS + Android

Examples: `ExpoHead.ios.tsx`, `NativeTabsView.web.tsx`, `useBackButton.native.ts`

// TODO(@ubax): add section about native components
// TODO(@ubax): add section about router-e2e
// TODO(@ubax): add verification for each feature - test, lint, build

