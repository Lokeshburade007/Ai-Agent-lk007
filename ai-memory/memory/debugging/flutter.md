# Flutter Debugging Playbook

## "RenderFlex overflowed by X pixels"

Yellow/black stripes on screen. A `Row` or `Column` child wanted more space than was available.

Fixes:
- Wrap the offending child in `Expanded` (takes remaining space) or `Flexible` (can shrink).
- Use `SingleChildScrollView` if content legitimately exceeds the screen.
- Use `Wrap` instead of `Row` for elements that should flow to multiple lines.

## Hot reload doesn't reflect a change

- You modified `main()`, a const constructor, an enum, or a top-level field. Hot reload doesn't handle these. **Hot restart** (`R`) instead.
- You modified native code (iOS/Android). Full rebuild needed (`q` then `flutter run`).
- The widget you changed isn't in the current tree. Navigate to a screen that rebuilds it.

## "setState() called after dispose"

You called `setState` from an async callback that completed after the widget was disposed.

Fix:
```dart
if (!mounted) return;
setState(() => ...);
```

Better: use a `StateNotifier` / Riverpod `Notifier` — they handle disposal cleanly and don't tie state to the widget lifecycle.

## "BuildContext across async gaps"

```dart
final result = await someAsyncCall();
Navigator.of(context).pop(result);   // ⚠️ context might be invalid
```

Lint warns about this. Fix:
```dart
final result = await someAsyncCall();
if (!mounted) return;
Navigator.of(context).pop(result);
```

## Riverpod provider returns stale data

- You used `ref.read` where you needed `ref.watch`. `read` is a one-time read; `watch` subscribes to changes.
- The provider isn't being invalidated. After a mutation, call `ref.invalidate(providerName)`.
- `family` providers cache per-argument. Same argument → cached result. Use `ref.invalidate(provider(arg))` to refresh.

## Widget rebuilds too often — find why

```dart
import 'package:flutter/rendering.dart';

void main() {
  debugRepaintRainbowEnabled = true;  // visualizes repaints
  runApp(const MyApp());
}
```

Common causes:
- Parent rebuilds → all children rebuild. Use `const` constructors where possible.
- `Provider.of` / `context.watch` higher up than needed — move it deeper.
- `setState` in a parent when only one child needs the new state.

## Async function never completes / hangs

- Missing `await` somewhere — the function returns immediately.
- Unhandled exception silently failed. Wrap with try/catch and log.
- `Future` chained without `.then` and result discarded.

```dart
final result = await someFuture();   // ✅
someFuture().then((r) => print(r));  // works but easy to lose errors
```

## "Bad state: No element"

`firstWhere` / `first` on an empty iterable. Use `firstWhereOrNull` (collection package) or check `isNotEmpty` first.

## App crashes only on release build

`flutter run --release` to reproduce locally.

Common causes:
- `assert(...)` was carrying real logic. Asserts are stripped in release.
- Code relies on `kDebugMode` behavior.
- Tree-shaking removed code referenced by reflection.
- Native code mismatch — Pod install / Gradle sync needed.

## ImagePicker / Camera / Permissions silently fail

- Check `Info.plist` (iOS) / `AndroidManifest.xml` permissions.
- Request permission at runtime: `permission_handler` package.
- On iOS simulator, camera plugin works differently from device. Test on real device.

## Network requests fail with "Cleartext HTTP traffic not permitted"

Android 9+ blocks plaintext HTTP by default. For dev against localhost:
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<application android:usesCleartextTraffic="true" ...>
```

iOS: add `NSAppTransportSecurity` in `Info.plist`. **Never** ship a release build with HTTP allowed in production.

## Gradle build fails after Flutter upgrade

```bash
cd android && ./gradlew clean
cd .. && flutter clean
flutter pub get
flutter run
```

If still broken, check `android/build.gradle` and `android/app/build.gradle` for outdated plugin versions. `flutter doctor` will sometimes point to a Gradle/Kotlin mismatch.

## iOS CocoaPods errors after `flutter pub get`

```bash
cd ios
pod repo update
pod install --repo-update
cd ..
flutter run
```

If you switch Flutter SDK versions: `rm -rf ios/Pods ios/Podfile.lock && cd ios && pod install`.

## "ProcessException: signal 9" / out of memory

The device killed the app. Memory leak or oversized assets. Use DevTools memory profiler to find growth. Common: image cache not bounded — set `PaintingBinding.instance.imageCache.maximumSizeBytes`.
