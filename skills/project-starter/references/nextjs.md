# Next.js Setup Reference

Use this reference only after deciding that a React frontend needs backend behavior. For Next.js implementation details, route to the `vercel-react-best-practices` skill first; this file only covers project-starter-specific scaffolding choices.

## When to Choose Next.js

- The frontend needs API routes, server actions, middleware, or server-rendered routes.
- There is no existing backend API and the project needs a small backend surface.
- Routes need to call AI inference, payment, database, or third-party APIs without exposing secrets to the browser.
- Deployment is expected to target Vercel or another platform with first-class Next.js support.

Use Vite + React instead when the app is mostly client-side and already has a backend API.

## Dependencies

Verify current stable versions first:

```bash
for package in next react react-dom typescript @types/node @types/react @types/react-dom eslint prettier; do
  npm view "$package" version
done
```

Then scaffold with the verified current Next.js CLI or create the files directly if the user needs a custom structure:

```bash
npx create-next-app@latest my-app --ts --eslint --app --src-dir
```

## Secret Handling

- Keep AI inference keys and third-party service credentials in server-only environment variables.
- Do not expose secret values through `NEXT_PUBLIC_*`.
- Put browser-callable wrappers in route handlers or server actions.
- Include `.env.example` with names only, not real values.

## Required Project-Starter Additions

- Add `AGENTS.md` with stack, package-manager, dev/build/test commands, environment rules, and the instruction to use `vercel-react-best-practices` for Next.js and React implementation.
- Add README setup instructions for install, dev, build, lint, typecheck, test, and environment variables.
- Verify versions again after `create-next-app`, because CLIs can generate package versions that lag the registry.
