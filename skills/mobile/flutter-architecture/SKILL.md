---
name: flutter-architecture
description: "Use when creating, scaffolding, structuring, or incrementally migrating Flutter projects with Riverpod (codegen) + hooks_riverpod/flutter_hooks, go_router, dio, freezed 3.0 + fpdart (Either), feature-based architecture by default, and Clean Architecture as an opt-in tier."
---

# Flutter complejo — patrón de proyecto

Caso principal: app Flutter consumiendo **REST propio vía dio** (mismo espíritu que react-architecture/next-architecture/laravel-architecture del otro lado — mismo backend, distinto cliente).

## Stack decidido y fijo

| Área | Elección |
|---|---|
| State management | Riverpod con codegen (`@riverpod`, `riverpod_generator`) — `Notifier`/`AsyncNotifier` reemplazan `StateNotifier` |
| Hooks | `hooks_riverpod` + `flutter_hooks` — `HookConsumerWidget`, `useState`/`useTextEditingController` en vez de `StatefulWidget` para estado local efímero |
| Navegación | go_router, auth guard vía `redirect` + `refreshListenable` |
| HTTP client | dio, interceptor de auth con refresh automático (cola con `Completer`, evita refresh concurrentes) |
| Modelos/serialización | freezed 3.0 + json_serializable (codegen, `build_runner`) — sintaxis nueva: `sealed class`/`abstract class` directo, sin `@freezed`, sin `.map()`/`.when()` (usar pattern matching de Dart) |
| Manejo de errores | `Either<Failure, T>` (fpdart) en repositories — nunca excepción al caller, `Failure` es un `sealed class` freezed |
| DI | Riverpod providers como DI primario; `get_it` solo si hace falta resolver algo fuera del árbol de widgets |
| Storage local | `shared_preferences` (prefs) + `flutter_secure_storage` (tokens, separado) por default; Hive/Drift solo si el proyecto necesita cache offline estructurado |
| Env config | `--dart-define` / `--dart-define-from-file`, sin paquete extra |
| Linting | `flutter_lints` (oficial) |
| Testing | `flutter_test` + `mocktail` (unit + widget) siempre; `integration_test` solo para flujos críticos; golden tests opcional, no default |
| Arquitectura | Feature-based simple por default. Clean Architecture (`domain/` con usecases) como tier opcional para proyectos grandes — ver `references/clean-architecture.md` |

No volver a preguntar estas decisiones — ya están fijadas para este patrón.

## Args (invocación `/flutter-architecture <texto>`)

Texto libre después del nombre del skill llega como bloque `ARGUMENTS` — no hay parser, se interpreta por convención. Claves reconocidas:

| Clave | Valores | Efecto |
|---|---|---|
| `nombre` / `name` | texto libre | Nombre/paquete del proyecto para `flutter create` |
| `arquitectura` | `simple` (default) / `clean` | `clean` aplica el tier de `references/clean-architecture.md` (domain/ con usecases) desde el arranque |
| `backend` | `rest` (default) / `firebase` / `supabase` | Si no es `rest`, avisar que `dio_client.dart`/`auth_repository.dart` no aplican tal cual |
| `integration_test` | `si` / `no` | Fuerza la decisión de incluir `integration_test/` en vez de preguntar (default: solo si hay flujo crítico) |
| `existente` | (sin valor) | Diagnosticar delta contra un proyecto Flutter ya existente en vez de scaffoldear desde cero |

Si `ARGUMENTS` no trae estas claves, seguir el flujo normal: preguntar lo que falte.

## Reglas fijas (no negociables dentro de este patrón)

1. **Repositories devuelven `Either<Failure, T>`, nunca tiran excepción al caller** (salvo dentro de `AsyncNotifier.build()`, donde Riverpod ya envuelve el `throw` en `AsyncError` — ahí sí se usa `throw failure` a propósito).
2. **`Failure` es un tipo cerrado (`sealed class`)** — el manejo en UI usa pattern matching de Dart (`switch`), nunca `if (failure is X)` en cadena.
3. **Un solo `Dio` instance** (`core/network/dio_client.dart`), interceptor de auth centralizado ahí. Ningún repository crea su propio `Dio`.
4. **Access + refresh token SIEMPRE en `flutter_secure_storage`**, nunca en `shared_preferences` (eso es solo para prefs no sensibles).
5. **`go_router` necesita el adapter `GoRouterRefreshNotifier`** para reaccionar a cambios de sesión — `refreshListenable` no soporta `AsyncValue` de Riverpod directo, solo `Listenable`/`ChangeNotifier`.
6. **`hooks_riverpod` para estado local efímero**, no `StatefulWidget` — mismo criterio que "no local component state innecesario" en los patrones React.
7. **`build_runner` corre después de cualquier cambio a un modelo/provider anotado** (`@riverpod`, `freezed`, `json_serializable`) — `dart run build_runner build --delete-conflicting-outputs`.
8. **`integration_test/` solo para flujos críticos** (login, checkout, pago) — no es parte del scaffold base salvo que se confirme.
9. **Límites de código.** `core/` contiene infraestructura reutilizable por app o múltiples features. Código específico queda dentro de `features/<feature>/`; features no se importan entre sí.
10. **Nombres y tests.** Archivos y carpetas Dart usan `snake_case`. Tests unit/widget reflejan la ruta de `lib/` bajo `test/` y terminan en `_test.dart`; no mezclar tests de integración con unit/widget.

## Verificación de versiones/APIs actuales

El ecosistema Flutter/Riverpod/freezed cambia rápido entre majors — lo de arriba (Riverpod 3, sintaxis Freezed 3.0, limitación de `refreshListenable`) se verificó contra fuentes actuales al armar este skill (2026-07, links en `references/scaffold.md`), no solo memoria de entrenamiento. Antes de asumir sintaxis no documentada acá, verificar primero con **ctx7** (`npx ctx7@latest library "<paquete>" "<query>"` seguido de `npx ctx7@latest docs <id> "<query>"`, o invocar el skill `find-docs`). Si ctx7 falla por cuota agotada, usar **WebSearch automático como fallback por default** — no parar a preguntarle al usuario cómo seguir, continuar con WebSearch y avisar en la respuesta que se usó como fallback. Correr `flutter --version` contra el proyecto real antes de asumir Dart 3.x/Freezed 3.0 — si es una versión menor, usar sintaxis `@freezed`/`.when()` en vez de `sealed class`/pattern matching.

## Estructura, resources y pasos de scaffold

Detalle completo (árbol de carpetas del tier simple, límites `core`/feature, convención de nombres y testing, tabla de forge resource ids, pasos numerados de scaffold, fuentes citadas) vive en `references/scaffold.md`. El tier Clean Architecture opcional vive aparte en `references/clean-architecture.md` — leer solo si el proyecto lo pidió. Ninguno hace falta para solo responder preguntas conceptuales sobre el patrón.
