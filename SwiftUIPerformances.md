# SwiftUI Performance Best Practices

## 1. The Mental Model: "Re-render" ≠ "Redraw"

When a `@State` / `@Observable` / `@Binding` dependency changes, SwiftUI re-invokes that view's `body` (relatively cheap — it's just building a value-type description of the view tree). Then it **diffs** the new tree against the old one and only updates the actual rendered output where things actually changed.

So the real performance goals are:

- Minimize how often `body` gets re-evaluated for big/expensive views
- Minimize the **size** of what gets re-evaluated each time
- Avoid breaking the diffing algorithm so it can skip unnecessary work

---

## 2. Use `@Observable`, Not `ObservableObject`

This is the single biggest win.

With `ObservableObject` + `@Published`, **any** change to **any** published property invalidates **every view** that reads the object — even if that view doesn't read the changed property.

```swift
// Old way - ANY @Published change re-renders ALL views observing this
class AlbumsViewModel: ObservableObject {
    @Published var albums: [Album] = []
    @Published var searchText: String = ""
    @Published var isLoading: Bool = false
}
```

```swift
// @Observable - tracks PER-PROPERTY access
@Observable
class AlbumsViewModel {
    var albums: [Album] = []
    var searchText: String = ""
    var isLoading: Bool = false
}
```

With `@Observable`, if `AlbumRow` only reads `album.title`, it only re-renders when *that property* changes — not when `isLoading` flips. This alone fixes a huge chunk of "why is everything re-rendering" problems.

---

## 3. Extract Subviews Aggressively

Every view has its own dependency graph. A giant `body` means SwiftUI has to re-diff a huge tree every time *any* dependency in it changes.

```swift
// Bad - whole list re-renders body when ANY album changes
var body: some View {
    List(albums) { album in
        HStack {
            AsyncImage(url: album.artworkURL)
            VStack {
                Text(album.title)
                Text(album.artist)
            }
            if album.isFavorite {
                Image(systemName: "heart.fill")
            }
        }
    }
}
```

```swift
// Good - extracted row is its own dependency unit
var body: some View {
    List(albums) { album in
        AlbumRow(album: album)
    }
}

struct AlbumRow: View {
    let album: Album

    var body: some View {
        HStack {
            AsyncImage(url: album.artworkURL)
            VStack {
                Text(album.title)
                Text(album.artist)
            }
            if album.isFavorite {
                Image(systemName: "heart.fill")
            }
        }
    }
}
```

When you extract a view, SwiftUI can diff/update just that row instead of re-running the whole `List`'s body closure.

---

## 4. Don't Pass the Whole Observable Object When You Only Need a Slice

```swift
// Bad - this row now depends on the WHOLE viewModel
struct AlbumRow: View {
    var viewModel: AlbumsViewModel
    let album: Album

    var body: some View {
        Text(album.title)
            .foregroundColor(viewModel.isLoading ? .gray : .primary) // depends on isLoading too
    }
}
```

Pass only what's needed — a value, or a focused `Binding` / `@Bindable` to a specific sub-object. This keeps each view's dependency graph minimal.

---

## 5. `ForEach` / `List` — Use Stable, Real Identifiers

```swift
// Bad - index-based identity, causes full diff churn on reorder/insert
ForEach(0..<albums.count, id: \.self) { i in
    AlbumRow(album: albums[i])
}

// Good - Identifiable with stable IDs lets SwiftUI diff efficiently
ForEach(albums) { album in   // Album: Identifiable
    AlbumRow(album: album)
}
```

If `Album.id` changes when data updates (e.g. you're generating a new UUID on every fetch), every row gets treated as "new" → full re-render + loses scroll position/state.

---

## 6. Avoid `AnyView`

`AnyView` erases the view type, which **breaks SwiftUI's diffing** — it can't compare the old/new tree structurally anymore, so it tends to tear down and rebuild the subtree entirely.

```swift
// Bad
func makeContent() -> AnyView {
    if isPremium {
        return AnyView(PremiumAlbumView(album: album))
    } else {
        return AnyView(StandardAlbumView(album: album))
    }
}
```

```swift
// Good - use @ViewBuilder / opaque types, keeps view type identity
@ViewBuilder
func makeContent() -> some View {
    if isPremium {
        PremiumAlbumView(album: album)
    } else {
        StandardAlbumView(album: album)
    }
}
```

---

## 7. Be Careful With `.id()`

`.id()` forces SwiftUI to treat the view as a **brand new instance** when the id changes — destroying and recreating it (losing `@State`, re-running `.task` / `.onAppear`, re-running expensive init work).

```swift
// This destroys/recreates the whole detail view every time selectedAlbum changes
AlbumDetailView(album: selectedAlbum)
    .id(selectedAlbum.id)
```

Only use `.id()` when you *want* that reset behavior (e.g. resetting scroll position or `@State` when navigating to a different item). Otherwise, let normal property updates flow through naturally.

---

## 8. Lazy Containers for Big Lists

`LazyVStack` / `LazyHStack` / `LazyVGrid` only instantiate views as they become visible. A regular `VStack` inside a `ScrollView` builds **everything** up front.

```swift
ScrollView {
    LazyVStack {
        ForEach(albums) { album in
            AlbumRow(album: album)
        }
    }
}
```

Use `List` when you can — it's lazy by default and has more built-in optimizations than a hand-rolled `LazyVStack`.

---

## 9. Don't Do Heavy Work in `body`

`body` can be called many times — it must be cheap and side-effect-free.

```swift
// Bad - sorts/filters EVERY time body runs
var body: some View {
    List(albums.filter { $0.isFavorite }.sorted { $0.title < $1.title }) { album in
        AlbumRow(album: album)
    }
}
```

```swift
// Good - compute once via a computed property on the view model
var body: some View {
    List(viewModel.favoriteAlbumsSorted) { album in
        AlbumRow(album: album)
    }
}
```

For really expensive values, consider `@State` + `.task` / `.onChange` to compute once and cache the result, rather than recomputing on every `body` pass.

---

## 10. `.equatable()` for Expensive Leaf Views

If a view is expensive to render and its inputs are `Equatable`, you can tell SwiftUI to skip re-rendering if the input value hasn't actually changed (even if `body` got called):

```swift
struct WaveformView: View, Equatable {
    let samples: [Float]

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.samples == rhs.samples
    }

    var body: some View {
        // expensive Canvas drawing
    }
}

// usage
WaveformView(samples: samples)
    .equatable()
```

Niche tool, but great for things like waveforms, charts, complex `Canvas` / `Shape` views.

---

## 11. `drawingGroup()` for Complex Layered Graphics

Flattens a subtree into a single Metal-rendered layer — good for stacks of shapes/gradients/blurs that would otherwise each be composited separately.

Costly to use everywhere (extra render pass + memory), so only apply it to genuinely complex composited views (e.g. a custom chart with many overlapping shapes).

---

## 12. Debugging Tools — Actually Measure This Stuff

Don't guess — verify:

```swift
var body: some View {
    let _ = Self._printChanges() // prints WHY this view's body was called
    ...
}
```

And in **Instruments**: the **SwiftUI** instrument has a "View Body" track that shows exactly how often each view's body runs and how long it takes. Run your app through it — you'll often be surprised which view is re-rendering on every scroll/state change.

---

## TL;DR Checklist

- [ ] `@Observable` over `ObservableObject` for granular property-level tracking
- [ ] Extract subviews so dependency graphs stay small
- [ ] Pass minimal data down (not whole view models)
- [ ] Stable `Identifiable` ids in `ForEach`
- [ ] Avoid `AnyView`; use `.id()` deliberately
- [ ] Lazy stacks / `List` for big collections
- [ ] No heavy computation in `body`
- [ ] `.equatable()` for expensive leaf views
- [ ] `drawingGroup()` for complex composited graphics
- [ ] Profile with `_printChanges()` + Instruments SwiftUI instrument
