# Testing — react-architecture (Vite + TanStack Router)

Stack: **Vitest 4 + React Testing Library + jsdom + MSW**. Playwright no es default.

## Layout

```text
src/test/
  setup.ts       # jest-dom, MSW listen/reset/close, stubs de browser
  handlers.ts    # handlers MSW por defecto (p. ej. CSRF)
  server.ts      # setupServer(...handlers)
  render.tsx     # renderWithProviders: QueryClient + i18n + TooltipProvider
  api.ts         # API_URL / APP_ORIGIN
```

Tests colocados: `foo.tsx` + `foo.test.tsx`. `__tests__/` solo para clusters chicos (`utils/`, `lib/`).

## Interacciones y red

- `user-event`, no `fireEvent`.
- Services: MSW contra el cliente HTTP real. No `vi.mock` de axios salvo tests viejos que aún no se migran.
- `onUnhandledRequest: 'bypass'` hasta que la suite viva en MSW; `'error'` cuando ya no hay fugas.

## Qué testear

El test vale si fallaría cuando se rompe algo que el usuario o el API notan.

- Formularios: submit válido, inválido, pending, no doble submit, reset.
- Dialogs: open/close, labels de la acción, error de mutation, pending, éxito cierra.
- Create y edit por separado aunque compartan el form.
- Services: URL + envelope `{ status, message, data }`.
- Schemas Zod con reglas de UI.
- Utils/stores con ramas (fallback, legacy).
- Guards de ruta.

## Qué no testear

- Componentes generados en `components/ui/` (shadcn).
- Archivos generados (`routeTree.gen.ts`), tipos-only, barrels.
- Terceros: Stripe Elements, driver.js, Iconify, Recharts, i18next internals.
- Un `useQuery` que solo llama al service (se cubre service + componente).
- Reglas de negocio del API si hay backend con tests propios.
- Copy estático, snapshots de DOM grande, `className`, layout.
- Lookup tables sin ramas.
- `vi.mock` de `Button`/`Dialog` para asertar que el mock se llamó.

## Playwright

No se instala en el scaffold. Si el usuario lo pide **y** la app es crítica:

- Solo journeys que **dejan de tener sentido si mockeas la API** (cookie, CORS, build desplegado, backend real).
- Pocos, post-deploy o contra staging. No en cada MR como suite principal.
- Nunca `page.route` como sustituto de MSW: eso es un test de componente caro.
- Stripe iframe / 3DS: QA manual, no Playwright.

Vitest Browser Mode sigue early: no usarlo como e2e.
