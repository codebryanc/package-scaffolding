# UI Layer Reference

## Folder Structure

```
UI/
  bloc/
    <feature>_event.dart
    <feature>_state.dart
    <feature>_bloc.dart
  page/
    <feature>_page.dart
  widgets/
    <feature>_form_widget.dart   ← extract form/content here or divide in little widgets
```

## BLoC — Events

```dart
abstract class FeatureEvent {
  const FeatureEvent();
}

class DoSomethingEvent extends FeatureEvent {
  final String value;
  const DoSomethingEvent({required this.value});
}
```

## BLoC — States

```dart
abstract class FeatureState {
  const FeatureState();
}

class FeatureInitial extends FeatureState {
  const FeatureInitial();
}

class FeatureLoading extends FeatureState {
  const FeatureLoading();
}

class FeatureSuccess extends FeatureState {
  final SomeModel result;
  const FeatureSuccess({required this.result});
}
```

## BLoC — Bloc file

- NO try-catch inside event handlers — never catch exceptions in the BLoC
- Use the result pattern: define `result` at the start, assign it, emit at the end

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

import '../../application/<feature>_application.dart';
import '<feature>_event.dart';
import '<feature>_state.dart';

class FeatureBloc extends Bloc<FeatureEvent, FeatureState> {
  FeatureBloc({required this.applicationService}) : super(const FeatureInitial()) {
    on<DoSomethingEvent>(_onDoSomething);
  }

  final FeatureApplication applicationService;

  Future<void> _onDoSomething(DoSomethingEvent event, Emitter<FeatureState> emit) async {
    emit(const FeatureLoading());
    bool result = false;
    result = await applicationService.doSomething(event.value);
    emit(FeatureSuccess(result: result));
  }
}
```

## Page

  - Uses `BlocBuilder` to switch between states
  - Use the `BlocBuilder` with switch that `builder: (context, state) => switch (state) {` and every state must have a `// comment`
  - Avoid generic states like `_` — always name every state explicitly

```dart
BlocBuilder<FeatureBloc, FeatureState>(
  builder: (context, state) => switch (state) {
    // Loading
    FeatureLoading() => const Center(child: CircularProgressIndicator()),
    // Initial
    FeatureInitial() => const SizedBox.shrink(),
    // Success
    FeatureSuccess() => const YourContentWidget(),
  },
)
```

## Import ordering

Always two groups separated by one blank line:

```dart
// 1. Third-party packages
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// 2. Project-relative imports
import '../bloc/<feature>_bloc.dart';
import '../bloc/<feature>_event.dart';
```

# Methods order

- Dispose always must be and the end of the file
