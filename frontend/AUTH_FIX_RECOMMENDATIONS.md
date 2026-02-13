# Authentication Persistence Fix - Recommendations

**Bug:** BUG-001 - Auth Redirect on Page Reload  
**Priority:** P1 (High)  
**Date:** February 13, 2026

---

## Problem Summary

When a logged-in user refreshes the page, authentication state is lost despite being stored in localStorage. The user is redirected to the login page.

**Root Cause:**
Likely a timing issue where React Router evaluates routes before Zustand has fully restored the auth state from localStorage.

---

## Recommended Solutions

### Option 1: React Router Loader Function (RECOMMENDED) ⭐

**Difficulty:** Medium  
**Time:** 30-60 minutes  
**Success Rate:** High  
**Breaking Changes:** Minimal

**Implementation:**

1. **Upgrade to React Router v6.4+ Data APIs**

```tsx
// src/main.tsx (new approach)
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { useAuthStore } from './store/authStore';

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    loader: async () => {
      // This runs before route renders
      const { isAuthenticated } = useAuthStore.getState();
      return { isAuthenticated };
    },
    children: [
      {
        path: 'home',
        element: <HomePage />,
        loader: authLoader, // Check auth before loading
      },
      // ... other routes
    ]
  }
]);

// src/loaders/authLoader.ts
export async function authLoader() {
  const { isAuthenticated, user } = useAuthStore.getState();
  
  if (!isAuthenticated) {
    throw redirect('/');
  }
  
  if (!user?.onboardingCompleted) {
    throw redirect('/onboarding');
  }
  
  return { user };
}
```

**Pros:**
- Official React Router pattern
- Guaranteed to run before component renders
- Clean separation of concerns
- Type-safe with TypeScript

**Cons:**
- Requires some refactoring
- Changes routing structure
- Need to understand React Router data APIs

---

### Option 2: Layout-Based Auth Check (SIMPLE) 🟢

**Difficulty:** Easy  
**Time:** 15-30 minutes  
**Success Rate:** High  
**Breaking Changes:** None

**Implementation:**

```tsx
// src/components/layouts/AuthLayout.tsx
import { useEffect } from 'react';
import { Outlet, useNavigate } from 'react-router-dom';
import { useAuthStore } from '../../store/authStore';
import { LoadingSpinner } from '../common/LoadingSpinner';

export function AuthLayout({ requireOnboarding = false }) {
  const { isAuthenticated, user } = useAuthStore();
  const navigate = useNavigate();
  const [isChecking, setIsChecking] = useState(true);

  useEffect(() => {
    // Give store a moment to rehydrate
    const timer = setTimeout(() => {
      const { isAuthenticated, user } = useAuthStore.getState();
      
      if (!isAuthenticated) {
        navigate('/', { replace: true });
      } else if (requireOnboarding && !user?.onboardingCompleted) {
        navigate('/onboarding', { replace: true });
      } else {
        setIsChecking(false);
      }
    }, 50); // Small delay to ensure store is rehydrated

    return () => clearTimeout(timer);
  }, []);

  if (isChecking) {
    return <LoadingSpinner fullScreen />;
  }

  return <Outlet />;
}

// src/App.tsx
<Routes>
  <Route element={<AuthLayout />}>
    <Route path="/onboarding" element={<OnboardingPage />} />
  </Route>
  
  <Route element={<AuthLayout requireOnboarding />}>
    <Route path="/home" element={<HomePage />} />
    <Route path="/disease" element={<DiseaseDetectionPage />} />
    {/* ... other protected routes */}
  </Route>
</Routes>
```

**Pros:**
- Simple to implement
- No major refactoring needed
- Easy to understand
- Works with existing code

**Cons:**
- Brief flash/delay on page load (50ms)
- Not the most "correct" solution
- Relies on setTimeout (not ideal)

---

### Option 3: Hydration Callback (ADVANCED) 🟡

**Difficulty:** Hard  
**Time:** 60-90 minutes  
**Success Rate:** Medium  
**Breaking Changes:** Moderate

**Implementation:**

```tsx
// src/store/authStore.ts
export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      tokens: null,
      isAuthenticated: false,
      _hydrated: false, // Internal flag
      
      // ... other state and actions
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        user: state.user,
        tokens: state.tokens,
        isAuthenticated: state.isAuthenticated,
      }),
      onRehydrateStorage: () => (state) => {
        // This callback runs AFTER rehydration
        if (state) {
          state._hydrated = true;
        }
      },
    }
  )
);

// Hook to wait for hydration
export function useHydration() {
  const hydrated = useAuthStore((state) => state._hydrated);
  return hydrated;
}

// src/App.tsx
function AppRouter() {
  const hydrated = useHydration();
  
  if (!hydrated) {
    return <LoadingSpinner fullScreen text="Loading..." />;
  }
  
  // Rest of router logic...
}
```

**Pros:**
- Proper state management solution
- No setTimeout hacks
- Type-safe

**Cons:**
- Previous attempt caused infinite loading (needs careful implementation)
- More complex to debug
- Could introduce new bugs

**Warning:** This was attempted before and failed. Only try if you fully understand Zustand persist internals.

---

### Option 4: localStorage Direct Check (QUICK HACK) 🔴

**Difficulty:** Very Easy  
**Time:** 5-10 minutes  
**Success Rate:** Medium  
**Breaking Changes:** None

**Implementation:**

```tsx
// src/components/common/ProtectedRoute.tsx
export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ 
  children,
  requireOnboarding = false 
}) => {
  const { isAuthenticated, user } = useAuthStore();
  
  // Fallback: Check localStorage directly
  useEffect(() => {
    if (!isAuthenticated) {
      const stored = localStorage.getItem('auth-storage');
      if (stored) {
        try {
          const data = JSON.parse(stored);
          if (data.state?.isAuthenticated) {
            // Force store to recognize auth state
            useAuthStore.setState(data.state);
            return;
          }
        } catch (e) {
          // Invalid stored data, ignore
        }
      }
    }
  }, [isAuthenticated]);
  
  // Rest of component...
}
```

**Pros:**
- Very quick to implement
- Minimal code changes
- Direct approach

**Cons:**
- Hacky solution
- Bypasses Zustand abstractions
- Could cause state inconsistencies
- Not maintainable

**Use only as:** Emergency fix or temporary workaround

---

## Recommended Approach

### For Immediate Demo Fix: Option 2 (Layout-Based) 🟢

**Why:**
- Quick to implement (15-30 min)
- Low risk of breaking existing code
- Easy to test and verify
- Reliable for demo purposes

**Implementation Steps:**
1. Create `AuthLayout.tsx` component
2. Wrap protected routes with `<Route element={<AuthLayout />}>`
3. Test with page refresh
4. Verify no infinite loading

### For Production: Option 1 (Router Loader) ⭐

**Why:**
- Official React Router best practice
- Scalable and maintainable
- Type-safe and testable
- Future-proof

**Timeline:**
- After demo/hackathon
- When you have time for proper testing
- As part of v2 improvements

---

## Testing Plan

### Before Fix
1. Login to app
2. Navigate to /home
3. Press F5 (refresh)
4. **Current:** Redirected to login ❌

### After Fix
1. Login to app
2. Navigate to /home  
3. Press F5 (refresh)
4. **Expected:** Stay on /home ✅

### Additional Tests
- Browser close and reopen
- Multiple tabs
- Logout and login again
- Token expiration handling

---

## Fallback for Demo

If time is too short to implement any fix:

### Workaround 1: Demo Mode (EASIEST) ✅
```bash
# Frontend .env
VITE_DEMO_MODE=true
```
- Bypasses auth completely for demo
- No page refresh issues
- Perfect for hackathon judges

### Workaround 2: Just Don't Refresh 🤷
- Train yourself not to press F5
- Use navigation only
- Brief judges about the working demo

---

## Implementation Priority

| Option | Priority | When to Use |
|--------|----------|-------------|
| Demo Mode | **NOW** | For demo day |
| Layout-Based (Option 2) | **Pre-Demo** | If you have 30 minutes |
| Router Loader (Option 1) | **Post-Demo** | For production release |
| Hydration Callback (Option 3) | **Never** | Too risky, previous failure |
| localStorage Hack (Option 4) | **Emergency Only** | Last resort |

---

## Estimated Effort

| Task | Time | Risk |
|------|------|------|
| Layout-Based Fix | 30 min | Low |
| Testing | 15 min | - |
| Documentation | 10 min | - |
| **Total** | **55 min** | **Low** |

---

## Decision Matrix

Choose your fix based on:

```
Time Available     | Recommended Solution
-------------------|---------------------
< 30 minutes       | Demo Mode (no code change)
30-60 minutes      | Layout-Based (Option 2)
1-2 hours          | Router Loader (Option 1)
> 2 hours          | Router Loader + tests
```

---

## Next Steps

1. **Immediate (5 min):**
   - Set `VITE_DEMO_MODE=true` for demo safety

2. **Pre-Demo (30 min):**
   - Implement Option 2 (Layout-Based)
   - Test thoroughly
   - Keep demo mode as backup

3. **Post-Demo (2 hours):**
   - Implement Option 1 (Router Loader)
   - Add comprehensive tests
   - Document the solution

---

## References

- **Zustand Persist:** https://docs.pmnd.rs/zustand/integrations/persisting-store-data
- **React Router Loaders:** https://reactrouter.com/en/main/route/loader
- **Bug Tracking:** See `BUG_TRACKING.md` BUG-001

---

**Author:** Frontend Team  
**Date:** February 13, 2026  
**Status:** Recommendations Ready  
**Next Review:** Before demo
