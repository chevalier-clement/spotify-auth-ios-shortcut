# Spotify Auth — iOS Shortcut Bridge

A static GitHub Pages page that handles the **Spotify OAuth 2.0 PKCE flow** and passes the resulting access token back to an iOS Shortcut.

No server, no secret — all parameters are supplied by the caller at runtime.

---

## How it works

1. You open the page with your `client_id`, `shortcut_name`, and optionally `scope` as query parameters.
2. The page displays the list of requested Spotify permissions and asks you to confirm.
3. After confirmation, the page redirects you to Spotify's authorization screen.
4. After you approve, Spotify redirects back to this page with a `code`.
5. The page exchanges that code for an access token (PKCE — no client secret needed).
6. The page opens `shortcuts://run-shortcut?name=<shortcut_name>&input=text&text=<access_token>`, passing the token to your Shortcut.

---

## Usage

```
GET https://chevalier-clement.github.io/spotify-auth-ios-shortcut/?client_id=<CLIENT_ID>&shortcut_name=<SHORTCUT_NAME>
```

### Required parameters

| Parameter       | Type   | Description                                               |
|----------------|--------|-----------------------------------------------------------|
| `client_id`     | string | Your Spotify application Client ID                        |
| `shortcut_name` | string | The exact name of the iOS Shortcut to call with the token |

### Optional parameters

| Parameter | Type   | Default                                                                                     | Description                              |
|-----------|--------|---------------------------------------------------------------------------------------------|------------------------------------------|
| `scope`   | string | *(none — public data only)* | Space-separated list of [Spotify scopes](https://developer.spotify.com/documentation/web-api/concepts/scopes). If omitted, Spotify grants access to public data only. The page displays the requested scopes to the user before redirecting. |

---

## Example

```
https://chevalier-clement.github.io/spotify-auth-ios-shortcut/?client_id=YOUR_CLIENT_ID&shortcut_name=SpotifyCallback
```

With a custom scope:

```
https://chevalier-clement.github.io/spotify-auth-ios-shortcut/?client_id=YOUR_CLIENT_ID&shortcut_name=SpotifyCallback&scope=user-read-playback-state%20user-modify-playback-state
```

---

## Setup

> **Want stricter access control?** You can fork this repo, make it private, and host it yourself on a platform that supports authentication (e.g. GitHub Enterprise Cloud, which can require GitHub login before serving a Pages site, or Cloudflare Pages / Vercel with an access policy). This prevents anyone other than authorized users from initiating the OAuth flow with your `client_id`. Note: a standard GitHub private repo does **not** protect the deployed Pages site — the page remains publicly accessible regardless of repo visibility unless your plan explicitly supports authenticated Pages.

No deployment needed — the page is already hosted at:

```
https://chevalier-clement.github.io/spotify-auth-ios-shortcut/
```

### 1. Register your Spotify application

1. Go to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard).
2. Create an application (or use an existing one).
3. Under **Edit Settings → Redirect URIs**, add the following URL exactly:
   ```
   https://chevalier-clement.github.io/spotify-auth-ios-shortcut/
   ```
4. Copy the **Client ID** — this is the value you pass as `client_id`.

> No Client Secret is required. This page uses the PKCE flow, which is designed for public clients.

### 2. Configure your iOS Shortcut

Create a Shortcut named exactly as the value you pass in `shortcut_name`. Its input will be the raw Spotify access token as plain text.

---

## Security note

The `client_id` is inherently public in a PKCE flow — it appears in the Spotify authorization URL in the browser. There is no `client_secret` involved. What stays private is the short-lived access token, which is passed directly to your Shortcut and never stored.

Because `scope` is a caller-supplied parameter, anyone can construct a URL requesting arbitrary Spotify permissions. The confirmation screen is your safeguard: always review the listed scopes before clicking **Authorize with Spotify**. Spotify also displays the same list on its own consent page.
