# Forms with Server Functions, React Hook Form, and Zod

Use this reference when a feature adds a validated form or resource dialog.

## Choose the form boundary

| Form shape | Pattern |
| --- | --- |
| Simple form with progressive enhancement | Native `<form action={serverFunction}>`, server-side Zod validation, and `useActionState` for expected errors. |
| Complex controlled form | React Hook Form with `zodResolver`; submit through the form handler, then call the Server Function or feature mutation. Validate again on the server. |
| shadcn/ui form | React Hook Form plus the current `Field` composition. Generate `field` before wiring the form. |
| Resource with create/edit/delete actions | One self-contained dialog per action. Do not merge modes behind `mode` or `isEdit`. |

Use native form actions when they cover the interaction. Add React Hook Form for controlled primitives, complex client feedback, field arrays, or behavior that needs its API. Do not add both to a form without a concrete reason.

## Dependencies and primitives

Add these when the first validated form needs them and the project does not already have them:

```text
react-hook-form
@hookform/resolvers
zod
```

In an existing project, include dependencies in the scoped plan and ask before installing. With shadcn/ui, generate only the primitives the feature needs:

```text
shadcn add field
shadcn add input-group   # only for grouped controls
```

## Schema contract

Keep one schema module per resource under the owning feature:

```text
features/<feature>/schemas/<resource>.ts
```

```ts
import * as z from "zod";

export const bugReportSchema = z.object({
  title: z.string().min(5).max(32),
  description: z.string().min(20).max(100),
});

export type BugReportValues = z.infer<typeof bugReportSchema>;
```

- Never declare the schema inside a component.
- Infer form values from the schema. Do not maintain a parallel hand-written type.
- Share a schema between create and edit only while validation is identical. Give changed edit validation its own module.
- Use plain-language messages that can render through the project's field error component.
- Parse again in every Server Function or Route Handler. Client validation is UX, not a security or integrity boundary.
- Validate external responses when their shape is not already guaranteed by a trusted typed client.

## Native Server Function form

Keep simple actions on the native form path:

```tsx
"use client";

import { useActionState } from "react";
import { createBugReport } from "../actions/create-bug-report";

type FormState = {
  errors: { title?: string[]; description?: string[] };
  message: string;
};

const initialState: FormState = { errors: {}, message: "" };

export function BugReportForm() {
  const [state, formAction, pending] = useActionState(
    createBugReport,
    initialState,
  );

  return (
    <form action={formAction}>
      <label htmlFor="bug-title">Title</label>
      <input id="bug-title" name="title" required aria-invalid={!!state.errors.title} />
      {state.errors.title?.map((error) => <p key={error}>{error}</p>)}
      <label htmlFor="bug-description">Description</label>
      <textarea id="bug-description" name="description" required />
      <p aria-live="polite">{state.message}</p>
      <button type="submit" disabled={pending}>Create</button>
    </form>
  );
}
```

The Server Function must accept the `prevState` argument first when used with `useActionState`, parse `FormData`, authenticate, authorize, mutate, and return only the state the UI needs. Return expected validation or business errors. Throw unexpected failures so an error boundary or server error path can handle them.

Use `useFormStatus` in a child submit control when pending state belongs to the form. Keep messages accessible with `aria-live`, and connect field errors with `aria-invalid` and the correct field description.

## React Hook Form and shadcn/ui

Each dialog owns its fields, `useForm`, submit wiring, labels, pending state, mutation call, reset behavior, and close behavior:

```tsx
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { Controller, useForm } from "react-hook-form";
import { bugReportSchema, type BugReportValues } from "../schemas/bug-report";

export function BugReportCreateDialog() {
  const form = useForm<BugReportValues>({
    resolver: zodResolver(bugReportSchema),
    defaultValues: { title: "", description: "" },
  });

  function onSubmit(values: BugReportValues) {
    // Call the Server Function or feature mutation, then reset or close.
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <Controller
        name="title"
        control={form.control}
        render={({ field, fieldState }) => (
          <input {...field} aria-invalid={fieldState.invalid} />
        )}
      />
      <button type="submit" disabled={form.formState.isSubmitting}>Create</button>
    </form>
  );
}
```

When shadcn/ui is present:

- Wrap related controls in `FieldGroup` and each control in `Field`.
- Set `data-invalid` and `aria-invalid` from field state.
- Use `Controller` for controlled or composed primitives. Use `register` for native inputs when it is simpler.
- Render errors through `FieldError errors={[fieldState.error]}` and helper text through `FieldDescription`.
- Give a form a stable `id` when an outside submit or reset button targets it.
- Keep one `useForm` per dialog and reset with `form.reset()` when the flow requires it.
- Use the project's existing toast or notification system. Do not add one for a single form.

## Dialog ownership

```text
features/<feature>/components/
├── <resource>-create-dialog.tsx
├── <resource>-edit-dialog.tsx
└── <resource>-delete-dialog.tsx
```

- Duplicate create and edit fields when their defaults, labels, action, or lifecycle can diverge.
- Extract a presentational field group only when fields are identical, action-independent, and used by at least three dialogs. It must not own `useForm`.
- Test each dialog independently. Do not create a shared `<resource>-form.tsx` only to avoid small duplication.

## Server completion

- Revalidate the affected path or tag after a successful mutation when the UI needs fresh data.
- Redirect only after mutation and revalidation. `redirect` is framework control flow; do not rely on code after it.
- Return structured action state for expected errors. Keep exception details server-side.
- Never mutate data, cookies, or cache during render. Submit from a form action or event handler.
