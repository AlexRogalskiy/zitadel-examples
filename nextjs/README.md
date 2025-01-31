This is our Zitadel [Next.js](https://nextjs.org/) template. If shows how to authenticate as a user and retrieve user information from the OIDC endpoint.

## Getting Started

First, we start by creating a new NextJS app with `npx create-next-app`, which sets up everything automatically for you. To create a project, run:

```bash
npx create-next-app --typescript
# or
yarn create next-app --typescript
```

# Install Authentication library

To keep the template as easy as possible we use [next-auth](https://next-auth.js.org/) as our main authentication library. To install, run:

```bash
npm i next-auth
# or
yarn add next-auth
```

To run the app, type:

```bash
npm run dev
```

then open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

# Configuration

NextAuth.js exposes a REST API which is used by your client.
To setup your configuration, create a file called [...nextauth].tsx in `pages/api/auth`.

```ts
import NextAuth from "next-auth";

export default NextAuth({
  providers: [
    {
      id: "zitadel",
      name: "zitadel",
      type: "oauth",
      version: "2",
      wellKnown: process.env.ZITADEL_ISSUER,
      authorization: {
        params: {
          scope: "openid email profile",
        },
      },
      idToken: true,
      checks: ["pkce", "state"],
      client: {
        token_endpoint_auth_method: "none",
      },
      async profile(profile) {
        return {
          id: profile.sub,
          name: profile.name,
          firstName: profile.given_name,
          lastName: profile.family_name,
          email: profile.email,
          loginName: profile.preferred_username,
          image: profile.picture,
        };
      },
      clientId: process.env.ZITADEL_CLIENT_ID,
    },
  ],
});
```

We recommend using the Authentication Code flow secured by PKCE for the Authentication flow.
To be able to connect to ZITADEL, navigate to your [Console Projects](https://console.zitadel.ch/projects) create or select an existing project and add your app selecting WEB, then PKCE, and then add `http://localhost:3000/api/auth/callback/zitadel` as redirect url to your app.

For simplicity reasons we set the default to the one that next-auth provides us. You'll be able to change the redirect later if you want to.

Hit Create, then in the detail view of your application make sure to enable dev mode. Dev mode ensures that you can start an auth flow from a non https endpoint for testing.

> Note that we get a clientId but no clientSecret because it is not needed for our authentication flow.

Now go to Token settings and check the checkbox for **User Info inside ID Token** to get your users name directly on authentication.

## Environment

Create a file `.env` in the root of the project and add the following keys to it.
You can find your Issuer Url on the application detail page in console.

```
NEXTAUTH_URL=http://localhost:3000
ZITADEL_ISSUER=[yourIssuerUrl]
ZITADEL_CLIENT_ID=[yourClientId]
```

# User interface

Now we can start editing the homepage by modifying `pages/index.tsx`.
Add this snippet your file. This code gets your auth session from next-auth, renders a Logout button if your authenticated or shows a Signup button if your not.
Note that signIn method requires the id of the provider we provided earlier, and provides a possibilty to add a callback url, Auth Next will redirect you to the specified route if logged in successfully.

```ts
import { signIn, signOut, useSession } from 'next-auth/client';

export default function Page() {
    const [session, loading] = useSession();
    ...
    {!session && <>
        Not signed in <br />
        <button onClick={() => signIn('zitadel', { callbackUrl: 'http://localhost:3000/profile' })}>
            Sign in
        </button>
    </>}
    {session && <>
        Signed in as {session.user.email} <br />
        <button onClick={() => signOut()}>Sign out</button>
    </>}
    ...
}
```

### Session state

To allow session state to be shared between pages - which improves performance, reduces network traffic and avoids component state changes while rendering - you can use the NextAuth.js Provider in `/pages/_app.tsx`.
Take a loot at the template `_app.tsx`.

```ts
import { Provider } from "next-auth/client";

function MyApp({ Component, pageProps }) {
  return (
    <Provider session={pageProps.session}>
      <Component {...pageProps} />
    </Provider>
  );
}

export default MyApp;
```

Last thing: create a `profile.tsx` in /pages which renders the callback page.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.
