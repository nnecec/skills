---
name: no-use-effect
description:
  Avoid most React `useEffect` usage. Use this when writing or reviewing
  components; prefer derived state, event handlers, query abstractions, and
  keyed remounts, and reserve `useEffect(effect, [])` for real mount/unmount
  sync.
metadata:
  version: '1.0'
---

# No `useEffect`

Avoid calling `useEffect` for routine component logic. Most uses are better
expressed as derived state, event handlers, data-fetching abstractions, keyed
remounts, or a narrowly scoped `useEffect(effect, [])` for true mount/unmount
integration.

## Quick Reference

- Lint rule: treat `useEffect` as restricted by default; `useEffect(effect, [])`
  is the only routine exception
- React docs: <https://react.dev/learn/you-might-not-need-an-effect>

| Instead of `useEffect` for...                 | Prefer                         |
| --------------------------------------------- | ------------------------------ |
| Deriving state from props or other state      | Inline computation             |
| Fetching data                                 | Query or data-fetching library |
| Responding to user actions                    | Event handlers                 |
| One-time external sync on mount               | `useEffect(effect, [])`        |
| Resetting state when an identity prop changes | `key` prop on a parent         |

## When To Use This Skill

- Writing new React components
- Refactoring components that already use `useEffect`
- Reviewing PRs that introduce `useEffect`
- Cleaning up defensive or cargo-cult React code

## Workflow

### 1. Identify why the effect exists

Classify the effect before changing it:

- Deriving view state from props or state
- Fetching remote data
- Relaying a user action
- Synchronizing with an external system
- Resetting component state when an identity changes

### 2. Replace it with the matching pattern

Use the five rules below. If none of them fit, pause and justify why an effect
is actually necessary.

### 3. Verify

Run the project checks that exist in the target repo, for example:

```bash
npm run lint -- --filter=<package>
npm run typecheck -- --filter=<package>
npm run test -- --filter=<package>
```

## Allowed Exception: `useEffect(effect, [])`

For the rare case where a component must synchronize with an external system
exactly once on mount, use `useEffect(effect, [])` directly:

```ts
import { useEffect } from 'react'

function VideoPlayer() {
  useEffect(() => {
    playVideo()
    return () => stopVideo()
  }, [])
}
```

If the repo bans `useEffect` outright, keep any lint suppression local to this
call and add a brief note explaining the external sync.

Use this only for mount/unmount lifecycle integration, not for general state
choreography.

## Replacement Patterns

### Rule 1: Derive state, do not sync it

If state can be calculated from props or other state, compute it during render
instead of storing a mirrored copy.

```tsx
// Bad: extra render just to mirror state
function ProductList() {
  const [products, setProducts] = useState([])
  const [filteredProducts, setFilteredProducts] = useState([])

  useEffect(() => {
    setFilteredProducts(products.filter(product => product.inStock))
  }, [products])
}
```

```tsx
// Good: derive inline in a single render
function ProductList() {
  const [products, setProducts] = useState([])
  const filteredProducts = products.filter(product => product.inStock)
}
```

Smell test: you are about to write `useEffect(() => setX(deriveFromY(y)), [y])`,
or you have state that only mirrors other props or state.

### Rule 2: Use data-fetching libraries

Fetching inside effects pushes caching, cancellation, retries, and stale-state
handling into component code.

```tsx
// Bad: effect-driven fetch with race-condition risk
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null)

  useEffect(() => {
    fetchProduct(productId).then(setProduct)
  }, [productId])
}
```

```tsx
// Good: query library handles lifecycle concerns
function ProductPage({ productId }) {
  const { data: product } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  })
}
```

Smell test: the effect does `fetch(...)` and then `setState(...)`, or it starts
re-implementing caching, retries, or cancellation.

### Rule 3: Use event handlers, not effects

If the user did something, perform the work directly in the event handler
instead of setting a flag and waiting for an effect to react.

```tsx
// Bad: state flag triggers side effect later
function LikeButton() {
  const [liked, setLiked] = useState(false)

  useEffect(() => {
    if (liked) {
      postLike()
      setLiked(false)
    }
  }, [liked])

  return <button onClick={() => setLiked(true)}>Like</button>
}
```

```tsx
// Good: action happens where the intent starts
function LikeButton() {
  return <button onClick={() => postLike()}>Like</button>
}
```

Smell test: the code reads like "set flag, effect wakes up, do real work, reset
flag."

### Rule 4: Use `useEffect(effect, [])` for one-time external sync

For DOM integration, browser APIs, third-party widgets, or singleton
subscriptions, prefer mounting the component only when prerequisites are ready
and then syncing once.

```tsx
// Bad: effect guards around readiness
function VideoPlayer({ isLoading }) {
  useEffect(() => {
    if (!isLoading) playVideo()
  }, [isLoading])
}
```

```tsx
// Good: only mount when ready, then sync once
function VideoPlayerWrapper({ isLoading }) {
  if (isLoading) return <LoadingScreen />
  return <VideoPlayer />
}

function VideoPlayer() {
  useEffect(() => playVideo(), [])
}
```

For stable singleton-like dependencies:

```tsx
useEffect(() => {
  connectionManager.on('connected', handleConnect)
  return () => connectionManager.off('connected', handleConnect)
}, [])
```

Smell test: the lifecycle is naturally "set up on mount, clean up on unmount."

### Rule 5: Reset with `key`, not dependency choreography

If a component should behave like a fresh instance when an identity prop
changes, remount it with a `key` instead of writing reset logic in an effect.

```tsx
// Bad: effect tries to emulate remount behavior
function VideoPlayer({ videoId }) {
  useEffect(() => {
    loadVideo(videoId)
  }, [videoId])
}
```

```tsx
// Good: keyed remount creates a clean instance
function VideoPlayer({ videoId }) {
  useEffect(() => {
    loadVideo(videoId)
  }, [])
}

function VideoPlayerWrapper({ videoId }) {
  return <VideoPlayer key={videoId} videoId={videoId} />
}
```

Smell test: the effect only exists to reset local state or restart behavior when
an ID changes.

## Review Checklist

When reviewing code, check for these questions in order:

1. Is this effect only deriving data that could be computed inline?
2. Is this effect fetching data that belongs in a query abstraction?
3. Is this effect reacting to an interaction that should stay in an event
   handler?
4. Is this a real mount/unmount sync case that justifies
   `useEffect(effect, [])`?
5. Would a keyed remount express the intent more clearly?

If the answer to all five is "no," require a concrete explanation for why the
effect is unavoidable.

## Component Structure Convention

Favor this order inside components:

1. Hooks
2. Local state
3. Derived values
4. Event handlers
5. Early returns
6. Render output

Computed values should be inline values, not `useEffect` plus `setState`.
