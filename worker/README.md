# DARKTEC Kevin API

This Cloudflare Worker provides the private server-side connection between the DARKTEC Evolution Kevin chat widget and the OpenAI Responses API.

## What it does

- Keeps `OPENAI_API_KEY` on the server as a Cloudflare secret.
- Accepts only a short, validated chat history.
- Uses the Kevin persona and the public DARKTEC project information.
- Sends requests to `POST https://api.openai.com/v1/responses` with `store: false`.
- Limits output size and applies a Cloudflare rate-limit binding (12 requests per 60 seconds per client IP, approximate and local to each Cloudflare location).
- Restricts browser origins to the GitHub Pages domain configured in `ALLOWED_ORIGINS`.
- Does not log message contents in this Worker code.

## Deploy with Wrangler

Use a computer with Node.js installed. From the repository root:

```bash
cd worker
npx wrangler@latest login
npx wrangler@latest secret put OPENAI_API_KEY
npx wrangler@latest deploy
```

When prompted for `OPENAI_API_KEY`, paste the API key created in OpenAI Platform. The key is entered into Wrangler's secure prompt; do not put it in a source file, GitHub commit, browser code, or chat.

The Worker URL will look like:

```
https://darktec-kevin-api.YOUR-CLOUDFLARE-SUBDOMAIN.workers.dev
```

The health check is:

```
https://darktec-kevin-api.YOUR-CLOUDFLARE-SUBDOMAIN.workers.dev/health
```

The chat endpoint is:

```
https://darktec-kevin-api.YOUR-CLOUDFLARE-SUBDOMAIN.workers.dev/api/chat
```

The Wrangler configuration declares the non-secret model and allows the repository's listed Vercel domain (`https://what-darktec.vercel.app`) plus the associated GitHub Pages domain. Change `ALLOWED_ORIGINS` if the website is moved to another domain. Change `OPENAI_MODEL` only to a model available to the OpenAI project key.

## Connect the website

After the Worker is deployed, edit the root `kevin-config.js` file and set `apiUrl` to the full chat endpoint:

```js
window.DARKTEC_KEVIN_CONFIG = Object.freeze({
  apiUrl: "https://darktec-kevin-api.YOUR-CLOUDFLARE-SUBDOMAIN.workers.dev/api/chat"
});
```

Commit that frontend configuration change to the site's deployment branch. The URL is public configuration; the OpenAI key is not.

Until `apiUrl` is set, Kevin stays in the existing demo mode with prepared answers. If the Worker is configured but unavailable, the widget displays a connection error rather than silently pretending that a demo answer came from AI.

## Cost and privacy notes

- OpenAI API usage is billed separately from a ChatGPT subscription. Set an appropriate budget and usage limits in the OpenAI Platform project.
- The Worker sets `store: false` for Responses API requests. This does not by itself disable OpenAI's default abuse-monitoring logs, which may retain API content for up to 30 days. Review the current [OpenAI API data controls](https://developers.openai.com/api/docs/guides/your-data).
- Cloudflare processes requests to operate the Worker. The rate limiter uses the request IP address as its visitor key. Review [Cloudflare's privacy policy](https://www.cloudflare.com/privacypolicy/).
- Before enabling the public chat, review and update the site's privacy notice, legal basis, provider details, and any required user notice or consent for the actual deployment.
- The rate limiting binding is approximate and location-local, not a strict global quota. It is an abuse-reduction measure, not authentication.
