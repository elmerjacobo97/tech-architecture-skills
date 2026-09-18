# Forms with shadcn/ui and Zod

Use this reference when a project uses shadcn/ui and a form needs validation.

## Dependencies and primitives

Validated forms are built on React Hook Form, `@hookform/resolvers`, and Zod. Install them when the first validated form is added and the project does not already have them:

```text
react-hook-form
@hookform/resolvers
zod
```

In an existing project, include them in the scoped plan and ask before installing. Generate the form primitives the feature needs:

```text
shadcn add field
shadcn add input-group   # only for grouped inputs (prefix/suffix, textarea with counter, block addons)
```

## Keep the schema in its own module

Validation schemas live outside the component, one module per resource, under the feature that owns it:

```text
features/<feature>/schemas/<resource>.ts
```

```ts
import * as z from "zod";

export const bugReportSchema = z.object({
  title: z
    .string()
    .min(5, "Bug title must be at least 5 characters.")
    .max(32, "Bug title must be at most 32 characters."),
  description: z
    .string()
    .min(20, "Description must be at least 20 characters.")
    .max(100, "Description must be at most 100 characters."),
});

export type BugReportValues = z.infer<typeof bugReportSchema>;
```

- Never declare the schema inside the component.
- One schema module per resource; reuse it between create and edit when the fields match.
- Infer form values from the schema; do not hand-write a parallel type.
- Write messages in plain language; they render through `FieldError`.
- Match the project import style (`import * as z from "zod"` in shadcn examples, `import { z } from "zod"` elsewhere).
- Re-validate in the Server Action or Route Handler; client validation is not a security boundary.

## Component wiring

```tsx
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { Controller, useForm } from "react-hook-form";

import { Button } from "@/components/ui/button";
import {
  Field,
  FieldDescription,
  FieldError,
  FieldGroup,
  FieldLabel,
} from "@/components/ui/field";
import { Input } from "@/components/ui/input";

import { bugReportSchema, type BugReportValues } from "../schemas/bug-report";

export function BugReportForm() {
  const form = useForm<BugReportValues>({
    resolver: zodResolver(bugReportSchema),
    defaultValues: {
      title: "",
      description: "",
    },
  });

  function onSubmit(values: BugReportValues) {
    // Call the action or mutation with the parsed values.
  }

  return (
    <form id="bug-report-form" onSubmit={form.handleSubmit(onSubmit)}>
      <FieldGroup>
        <Controller
          name="title"
          control={form.control}
          render={({ field, fieldState }) => (
            <Field data-invalid={fieldState.invalid}>
              <FieldLabel htmlFor="bug-report-title">Bug title</FieldLabel>
              <Input
                {...field}
                id="bug-report-title"
                aria-invalid={fieldState.invalid}
                placeholder="Login button not working on mobile"
                autoComplete="off"
              />
              {fieldState.invalid && <FieldError errors={[fieldState.error]} />}
            </Field>
          )}
        />
      </FieldGroup>
    </form>
  );
}
```

Rules:

- Wrap related fields in `FieldGroup`; wrap each control in `Field` with `data-invalid` and, when disabled, `data-disabled`.
- Use `Controller` for composed primitives and controlled components; `register` remains valid for native inputs when it keeps the interface simpler.
- Set `aria-invalid={fieldState.invalid}` on the control.
- Render validation through `FieldError errors={[fieldState.error]}`; add `FieldDescription` for helper text.
- Submit through `form.handleSubmit`; never call the mutation directly from a click handler.
- Give the form a stable `id` when the submit or reset button lives outside it (`<Button type="submit" form="bug-report-form">`).
- Reset with `form.reset()` when the flow needs it; default values live in `useForm`.
- Keep one `useForm` per form component; split the component when the form grows past one responsibility.
- Toast or sonner integrations are project decisions; add them only when the project already has one.

## Verification

- Valid submit calls the action with the parsed values.
- Invalid submit renders the field error and does not call the action.
- Reset restores the default values.
- Disabled and pending states block duplicate submits.
