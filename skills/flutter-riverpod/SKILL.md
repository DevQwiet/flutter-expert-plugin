---
name: flutter-riverpod
description: >
  Expert Flutter 3.x + Riverpod + Supabase. Use when working on a Flutter app
  with Riverpod state management and/or Supabase backend. Triggers on:
  "flutter", "riverpod", "supabase flutter", "provider", "ConsumerWidget",
  "AsyncNotifier", "go_router", "Google Sign-In", "nonce", "share_plus".
---

# Flutter + Riverpod Expert

Apply the conventions below automatically when working on any Flutter project using Riverpod and/or Supabase Flutter.

## Stack assumptions
- Flutter 3.6+
- `riverpod` 2.x with codegen (`@riverpod`)
- `supabase_flutter` 2.x
- `go_router` 14.x
- `share_plus` 11.x
- `google_sign_in` 7.x (if auth is needed)

## Non-negotiable rules

### Opacity — withValues, never withOpacity
```dart
// ✅
color.withValues(alpha: 0.5)
// ❌ deprecated Flutter 3.27+
color.withOpacity(0.5)
```

### share_plus 11.x API
```dart
// ✅
SharePlus.instance.share(ShareParams(text: '...'))
// ❌ removed in 11.x
Share.share('...')
```

### DateTime to Supabase — always UTC
```dart
// ✅
datetime.toUtc().toIso8601String()
// ❌
DateTime.now().toIso8601String()
```

### BuildContext after await
```dart
// ✅
await someAsyncCall();
if (!context.mounted) return;
Navigator.of(context).pop();
// ❌ may crash if widget disposed
await someAsyncCall();
Navigator.of(context).pop();
```

## Riverpod conventions

**Always use codegen:**
```dart
// ✅
@riverpod
Future<List<Game>> games(Ref ref) async { ... }

// ❌ manual provider
final gamesProvider = FutureProvider<List<Game>>((ref) async { ... });
```

**ConsumerWidget > StatefulWidget:**
```dart
// ✅
class GameList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final games = ref.watch(gamesProvider);
    ...
  }
}
```

**Optimistic update pattern (AsyncNotifier only):**
```dart
// Set state optimistically BEFORE await
state = AsyncData(updatedValue);
try {
  await repository.update(updatedValue);
} catch (e, st) {
  state = AsyncError(e, st);
  state = AsyncData(previousValue); // rollback
}
```

**Capture context refs before Navigator.pop:**
```dart
// ✅ capture before pop
final messenger = ScaffoldMessenger.of(context);
final router = GoRouter.of(context);
final notifier = ref.read(myProvider.notifier);
Navigator.of(context).pop();
await notifier.doSomething(); // safe

// ❌ ref invalid after pop
Navigator.of(context).pop();
await ref.read(myProvider.notifier).doSomething(); // crash
```

**After critical mutations (auth, onboarding):**
```dart
ref.invalidate(userProvider);
await ref.read(userProvider.future); // wait for fresh value
context.go('/home');
```

## Google Sign-In 7.x nonce flow
```dart
// Step 1: raw nonce
final rawNonce = generateRandomString(32);
// Step 2: hash it
final hashedNonce = sha256.convert(utf8.encode(rawNonce)).toString();
// Step 3: initialize with hashed nonce
await GoogleSignIn.instance.initialize(
  clientId: Platform.isIOS ? Env.iosClientId : null,
  serverClientId: Env.serverClientId,
  nonce: hashedNonce,
);
// Step 4: sign in
final result = await GoogleSignIn.instance.authenticate();
// Step 5: pass rawNonce (NOT hashed) to Supabase
await supabase.auth.signInWithIdToken(
  provider: OAuthProvider.google,
  idToken: result.credential!.idToken,
  nonce: rawNonce,
);
// Note: accessToken no longer exists in 7.x
// Note: never fall back to signInWithOAuth for Google
```

## Widget best practices

```dart
// ✅ const constructors reduce rebuilds
const MyWidget({super.key});

// ✅ small private classes > helper methods
class _GameCard extends StatelessWidget { ... }
// ❌ method returning Widget misses const/key caching
Widget _buildGameCard() => ...;

// ✅ ListView.builder for long lists
ListView.builder(
  itemCount: items.length,
  itemBuilder: (ctx, i) => ItemWidget(items[i]),
)
// ❌ eager rendering kills scroll performance
ListView(children: items.map((e) => ItemWidget(e)).toList())
```

## Dart modern patterns
```dart
// Records for multiple returns
(String, int) getUserInfo() => (user.name, user.age);

// Pattern matching over if-else chains
switch (result) {
  case Success(:final data) => renderData(data),
  case Error(:final message) => renderError(message),
}

// Null safety — prefer ?. and ?? over !
final name = user?.profile?.displayName ?? 'Anonymous';
```

## Pre-delivery checklist
- [ ] `flutter analyze --no-pub` returns zero warnings
- [ ] No `.withOpacity()` in changed files
- [ ] No `Share.share(...)` (use `ShareParams`)
- [ ] Every `await` followed by a `context.mounted` check before using `BuildContext`
- [ ] No `TODO`/`FIXME` left in delivered code
- [ ] `dart format` applied
