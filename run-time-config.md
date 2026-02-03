````md
# Nuxt Environment Variables: Best Practices

## Summary

In Nuxt 3 and 4, accessing `process.env` directly in application code is a common anti-pattern that leads to `undefined` errors, particularly in production or on the client side.

Nuxt uses a **Runtime Config** system to securely manage environment variables, ensuring they are properly hydrated and protected based on whether they are **public** or **private**.

---

## Lessons Learned

- **Security First**  
  Direct `process.env` usage can accidentally leak secrets to the browser. Runtime Config separates **public** (exposed to the browser) and **private** (server-only) keys.

- **Production Environment**  
  Nuxt does not read `.env` files in production. Variables must be defined in your hosting platform (Vercel, Docker, etc.).

- **Naming Convention**  
  Nuxt allows automatic overrides via environment variables. Variables prefixed with `NUXT_` in your `.env` or system environment will automatically map to your `runtimeConfig` keys.

- **Centralization**  
  All environment variables should be declared in `nuxt.config.ts` to provide a single source of truth and full TypeScript support.

---

## Corrected Example (Better-Auth)

### 1. Configure `nuxt.config.ts`

Define your expected variables here. This acts as a **schema** for your environment.

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Private (Server-only)
    betterAuthUrl: process.env.BETTER_AUTH_URL,
    googleClientSecret: process.env.GOOGLE_CLIENT_SECRET,

    public: {
      // Public (Available on Client & Server)
      googleClientId: process.env.GOOGLE_CLIENT_ID,
    },
  },
})
````

---

### 2. Implementation in Server Utilities

Use the `useRuntimeConfig()` composable to safely access these values in your Better-Auth setup.

```ts
// server/utils/auth.ts
import { betterAuth } from "better-auth";
import { mongodbAdapter } from "better-auth/adapters/mongodb";
import { client } from "@/utils/db";

// Access the safe Nuxt config
const config = useRuntimeConfig();

export const auth = betterAuth({
  database: mongodbAdapter(client.db()),
  baseURL: config.betterAuthUrl,
  emailAndPassword: {
    enabled: true,
  },
  socialProviders: {
    google: {
      // Access via public or private based on your nuxt.config.ts structure
      clientId: config.public.googleClientId,
      clientSecret: config.googleClientSecret,
      accessType: "offline",
      prompt: "select_account consent",
    },
  },
});
```

```
```
