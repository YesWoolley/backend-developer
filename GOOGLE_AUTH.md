# Google Auth Setup for Supabase

This guide explains how to enable **Google login** for a Supabase project.

---

# Google Auth Setup for Supabase

This guide explains how to enable Google login for a Supabase project.

## Overview

To use Google login with Supabase, configure both Google Cloud and Supabase.

High-level flow:

1. User clicks Continue with Google.
2. Google authenticates the user.
3. Google redirects to the Supabase callback URL.
4. Supabase creates the session.
5. Supabase redirects the user back to your frontend.

## 1. Create a Google Cloud Project

Go to https://console.cloud.google.com/ and create a new project (or use an existing one).

Example project name:

- Urban Intelligence Auth

## 2. Configure Consent Screen

In Google Cloud, go to either:

- Google Auth Platform
- APIs & Services -> OAuth consent screen

### Branding

Fill in:

- App name: your app name (example: Urban Intelligence)
- User support email: your email
- Developer contact email: your email

### Audience

Choose:

- External

Use External if normal public users should be able to sign in.

### Scopes

Add:

- openid
- .../auth/userinfo.email
- .../auth/userinfo.profile

These are standard scopes for Google login.

## 3. Create an OAuth Client ID

Go to:

- Google Auth Platform -> Clients
- Click Create Client

Choose:

- Application type: Web application

Example name:

- Supabase Google Login

## 4. Add Authorized JavaScript Origins

These should be frontend domains (origins only):

```text
https://urban-intelligence-dev.vercel.app
https://urban-intelligence.vercel.app
http://localhost:3000
```

Notes:

- Do not add paths here.
- Use origins only.
- Example: use `https://example.com`, not `https://example.com/login`.

## 5. Add Authorized Redirect URIs

These should be Supabase callback URLs, not frontend URLs.

Format:

```text
https://<project-ref>.supabase.co/auth/v1/callback
```

Example:

```text
https://nhqefzsctcqlwzosuwmx.supabase.co/auth/v1/callback
https://opogtjcpddbkhxsxkniq.supabase.co/auth/v1/callback
```

Important:

- If you have both dev and prod Supabase projects, add both callback URLs.

## 6. Copy Client ID and Client Secret

After creating the OAuth client, Google provides:

- Client ID
- Client Secret

Paste these into Supabase.

If you forget the secret:

1. Go to Google Cloud -> APIs & Services -> Credentials.
2. Open the OAuth client.
3. Copy the secret there (or reset it and then update Supabase).

## 7. Configure Google Provider in Supabase

In Supabase, go to Authentication -> Providers -> Google.

Then:

- Enable Google.
- Paste Client ID.
- Paste Client Secret.
- Keep the Callback URL as shown by Supabase.

Recommended settings:

- Skip nonce checks: OFF
- Allow users without email: OFF

## 8. Configure Supabase URL Settings

In each Supabase project, go to Authentication -> URL Configuration.

Dev Supabase:

- Site URL: `https://urban-intelligence-dev.vercel.app`
- Redirect URLs:
  - `https://urban-intelligence-dev.vercel.app/**`
  - `http://localhost:3000/**`

Production Supabase:

- Site URL: `https://urban-intelligence.vercel.app`
- Redirect URLs:
  - `https://urban-intelligence.vercel.app/**`

Important:

- Site URL = default frontend URL.
- Redirect URLs = allowed frontend routes after auth.
- These are different from Google callback URLs.

## 9. Frontend Login Code

Use Supabase client code like this:

```ts
await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: `${window.location.origin}/auth/callback`,
  },
})
```

## 10. Callback Route for Next.js

If you use SSR / PKCE flow, create `app/auth/callback/route.ts`:

```ts
import { NextResponse } from 'next/server'
import { createClient } from '@/utils/supabase/server'

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get('code')
  let next = searchParams.get('next') ?? '/'

  if (!next.startsWith('/')) {
    next = '/'
  }

  if (code) {
    const supabase = await createClient()
    const { error } = await supabase.auth.exchangeCodeForSession(code)

    if (!error) {
      return NextResponse.redirect(`${origin}${next}`)
    }
  }

  return NextResponse.redirect(`${origin}/auth/auth-code-error`)
}
```

## 11. Full Flow

```text
User
-> Frontend (Vercel / localhost)
-> Supabase signInWithOAuth()
-> Google login page
-> Google redirects to Supabase callback URL
-> Supabase creates session
-> Supabase redirects back to frontend
```

## 12. Callback URL vs Redirect URL

Google callback URL:

- Where Google sends the user after login.
- Example: `https://nhqefzsctcqlwzosuwmx.supabase.co/auth/v1/callback`
- Configure in Google Cloud -> Authorized redirect URIs.

Supabase redirect URL:

- Where Supabase sends the user after session creation.
- Example: `https://urban-intelligence.vercel.app/**`
- Configure in Supabase -> Authentication -> URL Configuration.

## 13. Common Mistakes

Mistake 1: Frontend URL used as Google redirect URI.

- Wrong: `https://urban-intelligence.vercel.app`
- Correct: `https://<project-ref>.supabase.co/auth/v1/callback`

Mistake 2: Missing Authorized JavaScript origins.

- Add:
  - `https://urban-intelligence-dev.vercel.app`
  - `https://urban-intelligence.vercel.app`
  - `http://localhost:3000`

Mistake 3: Frontend points to the wrong Supabase project.

- Ensure all of these belong to the same project:
  - `NEXT_PUBLIC_SUPABASE_URL`
  - Google callback URL
  - Supabase provider configuration

Mistake 4: Supabase redirect URLs too narrow (for example only `/login*`).

- Better: `https://urban-intelligence.vercel.app/**`
- OAuth often redirects to `/auth/callback` or other routes.

## 14. Recommended Environment Mapping

- Dev frontend: `https://urban-intelligence-dev.vercel.app`
- Prod frontend: `https://urban-intelligence.vercel.app`
- Dev Supabase callback: `https://nhqefzsctcqlwzosuwmx.supabase.co/auth/v1/callback`
- Prod Supabase callback: `https://opogtjcpddbkhxsxkniq.supabase.co/auth/v1/callback`

## 15. Final Checklist

Google Cloud:

- Project created
- Consent screen configured
- App audience set to External
- OAuth Client ID created
- Authorized JavaScript origins added
- Authorized redirect URIs added

Supabase:

- Google provider enabled
- Client ID pasted
- Client Secret pasted
- Site URL configured
- Redirect URLs configured

Frontend:

- `signInWithOAuth({ provider: 'google' })` added
- Callback route added
- Correct Supabase project environment variables used

## 16. Summary

For Google login with Supabase:

- Google Cloud: create OAuth client, add frontend origins, add Supabase callback URLs
- Supabase: enable Google provider, paste client ID and secret, configure Site URL and Redirect URLs
- Frontend: call `signInWithOAuth`, handle callback route, complete code exchange

This setup supports Google sign-in for both dev and production environments.