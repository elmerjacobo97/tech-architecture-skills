# Scaffold detallado — flutter-architecture

Leer este archivo solo cuando se va a scaffoldear de verdad. Stack, reglas y Args están en el SKILL.md principal. Si el proyecto pidió tier Clean Architecture, leer también `references/clean-architecture.md`.

## Estructura de carpetas (tier simple, default)

```
lib/
├── main.dart                    # ProviderScope + MaterialApp.router
├── app/
│   └── router/
│       └── app_router.dart      # go_router + GoRouterRefreshNotifier adapter
├── core/                         # infraestructura compartida; nunca importa features
│   ├── network/
│   │   └── dio_client.dart      # dio + interceptor auth/refresh (Completer queue)
│   ├── error/
│   │   └── failure.dart         # sealed Failure (freezed 3.0), retorno Either<Failure,T>
│   ├── storage/
│   │   └── secure_storage.dart  # flutter_secure_storage, solo tokens
│   └── env/
│       └── app_config.dart      # String.fromEnvironment, --dart-define
└── features/                     # slices verticales; no importación directa entre features
    ├── auth/
    │   ├── data/
    │   │   ├── models/user.dart
    │   │   └── repositories/auth_repository.dart
    │   └── presentation/
    │       ├── providers/auth_session_notifier.dart   # @riverpod AsyncNotifier<User?>
    │       └── screens/login_screen.dart
    └── <feature>/
        ├── data/
        │   ├── models/<model>.dart          # freezed 3.0 (abstract class + fromJson)
        │   └── repositories/<feature>_repository.dart
        └── presentation/
            ├── providers/<feature>_notifier.dart      # @riverpod AsyncNotifier
            ├── screens/<feature>_screen.dart           # HookConsumerWidget
            └── widgets/
test/
├── unit/
│   └── features/<feature>/<feature>_notifier_test.dart
└── widget/
    └── features/<feature>/<feature>_screen_test.dart
integration_test/                # SOLO si el flujo es critico (login, checkout, etc)
env/
├── dev.json                     # { "API_BASE_URL": "..." } — gitignored si trae secrets
└── prod.json
analysis_options.yaml
pubspec.yaml
```

`features/auth/` es un feature normal, mismo patrón que cualquier otro — su `auth_session_notifier.dart` es lo que escucha `app_router.dart` para los redirects.

## Convenciones de estructura y testing

- Usa `snake_case` en archivos y carpetas Dart: `auth_repository.dart`, `login_screen.dart`, `auth_session_notifier.dart`.
- Mantén infraestructura de aplicación o reutilizada por varios features en `core/`. No muevas código a `core/` solo por anticipar reutilización.
- Mantén cada slice vertical dentro de `features/<feature>/`. Un feature no importa otro feature directamente; extrae una abstracción a `core/` solo después de un segundo consumidor real.
- El tier simple mantiene `data/` y `presentation/` por feature. El tier Clean agrega `domain/` solo cuando fue confirmado; no mezclar tiers por defecto.
- Replica la ruta de producción bajo `test/`: `lib/features/posts/data/...` se prueba en `test/unit/features/posts/data/...`; widgets van en `test/widget/...`.
- Nombra tests con `_test.dart`. Deja `integration_test/` fuera de `test/` y úsalo solo para journeys críticos confirmados.

## Resources en forge

Traer con `forge-cli resource get <id> --json | jq -r '.content // .data.content'`.

| Archivo destino | forge resource id |
|---|---|
| `analysis_options.yaml` | `91a82309-cf76-4fe6-b4ba-597ad8806043` |
| Bloque `pubspec.yaml` (dependencies/dev_dependencies) | `213df1e9-3e4c-4526-b3f0-55519928a27f` |
| `lib/core/error/failure.dart` | `e24743fa-35f1-4df1-b9f7-6dfa0b107e11` |
| `lib/core/network/dio_client.dart` | `7d199984-d296-4956-8c9b-e1630066d0c6` |
| `lib/core/storage/secure_storage.dart` | `f9adec83-6d48-4e9d-8c66-0df731f35f06` |
| `lib/core/env/app_config.dart` | `4a083377-e0e1-4986-9fe2-6081623cc60b` |
| `lib/features/auth/presentation/providers/auth_session_notifier.dart` | `ba32ff37-6022-4b8f-909c-626ca82f41f4` |
| `lib/app/router/app_router.dart` | `593822bd-34f3-439d-909a-17fe9a66fdef` |
| `lib/main.dart` | `f078e446-3930-40dd-8d20-297272911cf4` |
| `lib/features/auth/data/models/user.dart` | `ad57ba34-d471-44c6-a19f-0f96fe6e8d36` |
| `lib/features/auth/data/repositories/auth_repository.dart` | `a78ed06f-d3f2-4419-ae20-4675cb941e31` |
| Template `lib/features/<feature>/data/models/<model>.dart` | `65adf437-3ed5-4924-9fda-eeefd901682a` |
| Template `lib/features/<feature>/data/repositories/<feature>_repository.dart` | `32f52213-c8a5-4cf0-856d-690d595aa898` |
| Template `lib/features/<feature>/presentation/providers/<feature>_notifier.dart` | `55f11a53-6038-4f17-a476-63dbe0c96746` |
| Template `lib/features/<feature>/presentation/screens/<feature>_screen.dart` | `8ad97532-9912-4c09-80bf-e46bd69b94df` |
| Ejemplo unit test (ProviderContainer + mocktail) | `5e76772e-1957-4397-8046-2d7f68d74bc6` |
| Ejemplo widget test (ProviderScope overrides) | `7b251a6b-6f1e-4a4f-bcaa-ae33e5ad8af8` |

## Pasos de scaffold (proyecto nuevo)

1. Confirmar: nombre del proyecto/paquete, tier (simple default vs Clean Architecture — ver `references/clean-architecture.md`), si necesita `integration_test` (flujo crítico), si el backend ya existe (react-architecture/next-architecture/laravel-architecture del otro lado, para alinear `API_BASE_URL`).
2. `flutter create <nombre> --org com.tuempresa`.
3. Verificar versión de Flutter/Dart: `flutter --version`. El stack asume Dart 3.x (sealed/pattern matching) y Freezed 3.0 — si el proyecto está en Dart <3 o Freezed 2.x instalado, avisar y usar sintaxis `@freezed`/`.when()`/`.map()` en vez de `sealed class`/pattern matching (ver nota en `failure.dart`).
4. Traer el bloque de `pubspec-deps.yaml` (resource de la tabla) y mergearlo a mano en `pubspec.yaml` (no sobreescribir el archivo generado por `flutter create`).
5. `flutter pub get`.
6. Crear la estructura de carpetas de la sección anterior.
7. Traer cada resource de la tabla y escribir al path destino, reemplazando `post`/`Post`/`posts` por el nombre real del primer feature.
8. Crear `env/dev.json` con `API_BASE_URL` real, agregar `env/*.json` a `.gitignore` si van a llevar valores sensibles (dejar `env/dev.json.example` sin secrets si hace falta compartir la forma).
9. Correr codegen: `dart run build_runner build --delete-conflicting-outputs` (genera `.freezed.dart`, `.g.dart` de freezed/json_serializable/riverpod_generator).
10. Reemplazar los placeholders `LoginScreen`/`HomeScreen` de `app_router.dart` por las screens reales.
11. Verificar: `flutter analyze`, `flutter test`.
12. Si se confirmó flujo crítico: `flutter test integration_test` (requiere emulador/device corriendo).

## Fuentes consultadas (WebSearch, ctx7 sin cuota ese día)

- [What's new in Riverpod 3.0](https://riverpod.dev/docs/whats_new) y [About code generation](https://riverpod.dev/docs/concepts/about_code_generation) — `@riverpod`, Notifier/AsyncNotifier unificados, autoDispose default con codegen
- [freezed changelog](https://pub.dev/packages/freezed/changelog) y [resumen de cambios 3.0](https://zenn.dev/dj_kusuha/articles/freezed_3_0_20250423?locale=en) — `sealed class`/`abstract class` reemplaza `@freezed`, remoción de `map`/`when`
- [Guarding routes in Flutter with GoRouter and Riverpod](https://dinkomarinac.dev/blog/guarding-routes-in-flutter-with-gorouter-and-riverpod/) — `refreshListenable` requiere `Listenable`, no soporta `AsyncValue` directo
- [Efficient Refresh Token Handling in Dio with Queued Interceptors](https://medium.com/@muhammad.kuifatieh/efficient-refresh-token-handling-in-dio-with-queued-interceptors-cc846dfdebf9) — patrón `Completer<void>` para evitar refresh concurrentes
- [fpdart Either fold/match](https://pub.dev/documentation/fpdart/latest/fpdart/Either-class.html) — `.fold()` y `.match()` son alias equivalentes

Si algo choca con la versión real instalada (Flutter/Dart/paquetes), `flutter --version` y `pubspec.lock` mandan sobre este archivo.
