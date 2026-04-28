# Common Layer Reference

## Responsibilities

- Global models shared across all layers
- Dependency injection setup
- Providers to expose BLoCs to the app

## Folder Structure

```
lib/common/
  models/
    <feature>/
      <feature>_model.dart
  di/
    dependency_injection.dart
  providers/
    btg_<package>_providers.dart
```

## Models

- No Equatable — never use `@override List<Object?> get props`
- Plain Dart classes with `const` constructor

```dart
class FeatureModel {
  const FeatureModel({required this.name});
  final String name;
}
```

## Dependency Injection

Always use `get_it` with exactly 4 fixed methods in this order:

```dart
import 'package:get_it/get_it.dart';

final GetIt getIt = GetIt.instance;

class DependencyInjection {
  static Future<void> init() async {
    await _initCore();
    await _initServices();
    await _initRepositories();
    await _businessLogic();
  }

  /// Register core dependencies (e.g. Dio)
  static Future<void> _initCore() async {
    getIt.registerLazySingleton<Dio>(() => Dio());
  }

  /// Register service dependencies
  static Future<void> _initServices() async {}

  /// Register repository dependencies (data sources, mapping, repository, domain, application)
  static Future<void> _initRepositories() async {
    getIt.registerLazySingleton<FeatureRepository>(
      () => FeatureRepositoryImpl(dio: getIt<Dio>()),
    );
  }

  /// Register business logic dependencies (BLoCs)
  static Future<void> _businessLogic() async {
    getIt.registerLazySingleton<FeatureBloc>(
      () => FeatureBloc(application: getIt<FeatureApplication>()),
    );
  }

  static void reset() => getIt.reset();
}
```

## Providers

Wraps all package BLoCs in a `MultiBlocProvider` so the app consumes them with a single widget:

```dart
class BtgPackageProviders extends StatelessWidget {
  const BtgPackageProviders({super.key, required this.child});

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        // Feature
        BlocProvider<FeatureBloc>(create: (_) => getIt<FeatureBloc>()),
      ],
      child: child,
    );
  }
}
```

## Barrel file

Export models, DI, and providers. Never export DTOs, data sources, mapping, or repository implementations:

```dart
// Common
export 'common/models/<feature>/<feature>_model.dart';
export 'common/di/dependency_injection.dart';
export 'common/providers/btg_<package>_providers.dart';

// Feature
export 'features/<feature>/UI/page/<feature>_page.dart';
export 'features/<feature>/UI/bloc/<feature>_bloc.dart';
export 'features/<feature>/UI/bloc/<feature>_event.dart';
export 'features/<feature>/UI/bloc/<feature>_state.dart';
```
