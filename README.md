# SSO Relay

A static page for GitHub Pages that receives SSO callbacks at a fixed public URL
and forwards them to a dev instance of FUSION IQ Web (localhost or a LAN IP).

The identity provider only redirects to URLs registered for the partner. Register
this page once, and you can test SSO from any machine without re-registering.

## How it works

A callback looks like this (UAT):

```
https://digitalinvestor-intg.navyfederal.org/app/#/login?SSORequestToken=…&SSORequestAttributes=…&PartnerName=NON_PROD/…
```

The relay keeps the route and parameters (`#/login?…`) and swaps the host and
path for the target root, which defaults to `http://localhost:9292/`:

```
http://localhost:9292/#/login?SSORequestToken=…&SSORequestAttributes=…&PartnerName=NON_PROD/…
```

The app's normal SSO handling on `/#/login` takes over from there. The backend
still validates the token, so the relay grants nothing by itself.

There are two ways in:

- **Replay (no IdP changes):** paste the callback URL into the page and it opens
  on the target. Use this when the partner's return URL is fixed to the UAT host.
- **Direct:** register this page as the partner's return URL. Callbacks are then
  forwarded automatically, to `RelayState` if it's allowed, otherwise to the
  saved target.

SSO parameters are read from both the query string and the hash.

Targets are limited to `localhost`, `*.local`, `127.*`, `10.*`, `172.16–31.*`
and `192.168.*`, so the page can't be used to send tokens to arbitrary sites.
To allow a public host, add it to `EXTRA_ALLOWED_HOSTS` in `index.html`.

## Setup

1. Push this folder to a GitHub repo and enable Pages
   (Settings → Pages → Deploy from branch → `main` / root).
2. Register `https://<user>.github.io/<repo>/` as the return URL for the SSO
   partner (for example `Local/Test_FusionIQ_PSP`).
3. If the IdP ignores `RelayState`, open this link once on each test device:
   `https://<user>.github.io/<repo>/?target=http://192.168.1.20:9292/`

## Limitations

- The IdP must return the parameters with a **GET** redirect. GitHub Pages can't
  accept a POST, so a form-post callback gets a 405 error.
- The saved target is per browser (localStorage on the `github.io` origin).
