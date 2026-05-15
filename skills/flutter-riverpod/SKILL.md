---
name: flutter-riverpod
description: >
  Expert Flutter 3.x + Riverpod avec standards d'accessibilité. Use when working
  on a Flutter app with Riverpod state management. Triggers on: "flutter",
  "riverpod", "ConsumerWidget", "AsyncNotifier", "provider", "go_router",
  "widget", "accessibility", "a11y", "semantics", "flutter analyzer".
---

# Flutter + Riverpod Expert

Enforce Flutter 3.x best practices, Riverpod codegen patterns, and WCAG accessibility standards.

## Non-negotiable rules

### Opacity — withValues only (Flutter 3.27+)
```dart
// ✅
color.withValues(alpha: 0.5)
// ❌ deprecated
color.withOpacity(0.5)
```

### BuildContext after await
```dart
// ✅
await doSomething();
if (!context.mounted) return;
Navigator.of(context).pop();
// ❌ widget may be disposed
```

### share_plus 11.x
```dart
// ✅
SharePlus.instance.share(ShareParams(text: '...'))
// ❌
Share.share('...')
```

## Riverpod

**Always codegen:**
```dart
// ✅
@riverpod
Future<List<Item>> items(Ref ref) async => repository.getAll();

// ❌ manual provider
final itemsProvider = FutureProvider((ref) => repository.getAll());
```

**ConsumerWidget over StatefulWidget:**
```dart
class ItemList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final items = ref.watch(itemsProvider);
    return items.when(
      data: (list) => ListView.builder(...),
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('$e'),
    );
  }
}
```

**Capture refs before Navigator.pop:**
```dart
// ✅
final notifier = ref.read(myProvider.notifier);
Navigator.of(context).pop();
await notifier.action(); // safe

// ❌ crash — ref disposed after pop
Navigator.of(context).pop();
await ref.read(myProvider.notifier).action();
```

**Optimistic update:**
```dart
final previous = state.requireValue;
state = AsyncData(optimistic);
try {
  await repo.save(optimistic);
} catch (e, st) {
  state = AsyncData(previous);
}
```

## Widget patterns

```dart
// ✅ const + class = free caching
class _ItemCard extends StatelessWidget {
  const _ItemCard({required this.item});
}

// ❌ method misses const/key
Widget _buildCard(Item item) => Card(...);

// ✅ lazy list
ListView.builder(itemCount: n, itemBuilder: (_, i) => _ItemCard(items[i]))

// ❌ eager — kills scroll perf beyond ~20 items
ListView(children: items.map(_ItemCard.new).toList())
```

## Accessibility (WCAG AA)

Every screen must pass:

| Rule | Requirement |
|------|-------------|
| Touch targets | ≥ 44×44 pt |
| Text contrast | 4.5:1 normal, 3:1 large (≥18pt) |
| Semantic labels | `Semantics(label:)` on icon-only buttons |
| Font scaling | UI intact at 200% system font size |
| Animation | Guard with `MediaQuery.of(context).disableAnimations` |
| Color-only info | Always add icon or text alongside color |

```dart
// Touch target
SizedBox(
  width: 44,
  height: 44,
  child: IconButton(icon: Icon(Icons.close), onPressed: onClose),
)

// Semantic label on icon button
Semantics(
  label: 'Close dialog',
  button: true,
  child: GestureDetector(onTap: onClose, child: Icon(Icons.close)),
)

// Decorative element
ExcludeSemantics(child: Icon(Icons.star))

// Animation guard
if (!MediaQuery.of(context).disableAnimations)
  AnimatedOpacity(...)
else
  child
```

## Dart modern patterns

```dart
// Records
(String name, int age) info() => (user.name, user.age);

// Exhaustive switch
switch (state) {
  case Loading() => CircularProgressIndicator(),
  case Error(:final message) => Text(message),
  case Data(:final value) => ContentWidget(value),
}

// Null safety
final label = item?.name ?? 'Unknown';
```

## Pre-delivery checklist
- [ ] `flutter analyze --no-pub` → zero warnings
- [ ] No `.withOpacity()` in changed files
- [ ] Every `await` + `BuildContext` has a `mounted` guard
- [ ] Touch targets ≥ 44pt on all tappable elements
- [ ] Icon-only buttons have `Semantics(label:)`
- [ ] Animations guarded with `disableAnimations`
- [ ] `dart format` applied
- [ ] No `TODO`/`FIXME` in delivered code
