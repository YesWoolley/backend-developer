# Azure Auth Setup for Supabase

This guide explains how to enable Microsoft (Azure AD / Entra ID) login for a Supabase project.

## Overview

To use Azure login with Supabase, configure both Microsoft Entra ID and Supabase.

High-level flow:

1. User clicks Continue with Microsoft.
2. Microsoft authenticates the user.
3. Microsoft redirects to the Supabase callback URL.
4. Supabase creates the session.
5. Supabase redirects the user back to your frontend.

## 1. Create or Select an Entra App Registration

Go to the Azure portal:

- https://portal.azure.com

Then open:

- Microsoft Entra ID -> App registrations

Create a new registration (or reuse an existing one).

Example app name:

- Urban Intelligence Auth

## 2. Configure Supported Account Types

When creating the app registration, choose one account type:

- Single tenant: only users in your organization
- Multitenant: users from any Microsoft Entra organization
- Multitenant + personal accounts: work/school + personal Microsoft accounts

Pick the one that matches your product requirements.

## 3. Add Redirect URI (Supabase Callback)

In the app registration, go to Authentication and add a Redirect URI.

Platform type:

- Web

Redirect URI format:

```text
https://<project-ref>.supabase.co/auth/v1/callback
```

Example:

```text
https://nhqefzsctcqlwzosuwmx.supabase.co/auth/v1/callback
https://opogtjcpddbkhxsxkniq.supabase.co/auth/v1/callback
```

Important:

- Use Supabase callback URLs, not frontend URLs.
- If you have dev and prod Supabase projects, add both callbacks.

## 4. Create Client Secret

In the app registration, go to:

- Certificates & secrets -> Client secrets -> New client secret

After creation, copy:

- Client secret Value (not Secret ID)

Also copy from Overview:

- Application (client) ID
- Directory (tenant) ID

You will use these values in Supabase.

## 5. Configure Azure Provider in Supabase

In Supabase, go to Authentication -> Providers -> Azure.

Fill in:

- Enabled: ON
- Client ID: Application (client) ID from Entra
- Secret: Client secret Value
- URL / Tenant: choose the Tenant URL that matches your sign-in audience

Keep the callback URL shown by Supabase unchanged.

## 5.1 Azure Tenant URL (for Supabase Azure Auth)

The Tenant URL tells Microsoft which directory of users is allowed to sign in.

Format:

```text
https://login.microsoftonline.com/<tenant>
```

Option 1 - common (most flexible):

```text
https://login.microsoftonline.com/common
```

Allows:

- Personal Microsoft accounts
- Outlook.com
- Hotmail.com
- Work accounts (Microsoft 365)
- Azure AD organization accounts

Example users:

- user@outlook.com
- user@hotmail.com
- user@company.com
- user@university.edu

Best for public applications.

Option 2 - consumers:

```text
https://login.microsoftonline.com/consumers
```

Allows only personal Microsoft accounts (for example outlook.com, hotmail.com, live.com).

Work accounts cannot sign in.

Option 3 - specific tenant ID:

```text
https://login.microsoftonline.com/65e6fd84-e889-4e52-8dc6-bde28e557b84
```

Allows only users from your organization (for example `@urbanintelligence.com.au`).

External Microsoft accounts cannot sign in.

### What You Should Use

For a public SaaS-style Urban Intelligence platform, use:

```text
https://login.microsoftonline.com/common
```

This supports sign-in from Outlook, Hotmail, Microsoft 365, and company accounts.

## 6. Configure Supabase URL Settings

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
- These are different from Entra redirect URIs.

## 7. Frontend Login Code

Use Supabase client code like this:

```ts
await supabase.auth.signInWithOAuth({
  provider: 'azure',
  options: {
    redirectTo: `${window.location.origin}/auth/callback`,
  },
})
```

## 8. Callback Route for Next.js

If you use SSR / PKCE flow, create app/auth/callback/route.ts:

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

## 9. Full Flow

```text
User
-> Frontend (Vercel / localhost)
-> Supabase signInWithOAuth()
-> Microsoft login page
-> Microsoft redirects to Supabase callback URL
-> Supabase creates session
-> Supabase redirects back to frontend
```

## 10. Callback URL vs Redirect URL

Entra redirect URI:

- Where Microsoft sends the user after login.
- Example: `https://nhqefzsctcqlwzosuwmx.supabase.co/auth/v1/callback`
- Configure in Entra App Registration -> Authentication.

Supabase redirect URL:

- Where Supabase sends the user after session creation.
- Example: `https://urban-intelligence.vercel.app/**`
- Configure in Supabase -> Authentication -> URL Configuration.

## 11. Common Mistakes

Mistake 1: Frontend URL used as Entra redirect URI.

- Wrong: `https://urban-intelligence.vercel.app`
- Correct: `https://<project-ref>.supabase.co/auth/v1/callback`

Mistake 2: Using Secret ID instead of Secret Value.

- In Supabase, always paste the client secret Value.

Mistake 3: Wrong tenant configuration.

- Ensure tenant/account type in Entra matches your intended user audience.

Mistake 4: Frontend points to the wrong Supabase project.

- Ensure these all match the same environment:
  - NEXT_PUBLIC_SUPABASE_URL
  - Supabase Azure provider config
  - Entra redirect URI

## 12. Final Checklist

Microsoft Entra ID:

- App registration created
- Supported account type selected
- Redirect URIs added (Supabase callback URLs)
- Client secret created and Value copied
- Client ID and Tenant ID copied
- Required permissions configured

Supabase:

- Azure provider enabled
- Client ID pasted
- Secret pasted
- Tenant URL configured (recommended for public apps: `https://login.microsoftonline.com/common`)
- Site URL configured
- Redirect URLs configured

Frontend:

- signInWithOAuth({ provider: 'azure' }) added
- Callback route added
- Correct Supabase environment variables used

## 13. Summary

For Azure login with Supabase:

- Entra ID: create app registration, set account type, add Supabase callback URLs, generate secret
- Supabase: enable Azure provider, paste client data, configure Site URL and Redirect URLs
- Frontend: call signInWithOAuth, handle callback route, complete code exchange

This setup supports Microsoft sign-in for both development and production environments.
