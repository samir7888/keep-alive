# Render Keep-Alive

A tiny GitHub Actions workflow that sends a `GET` request to your Render backend's health-check endpoint every 10 minutes. It uses only `curl` on an Ubuntu runner: no Python, Node.js, Docker, or other dependencies.

> **Important disclaimers**
> - GitHub Actions scheduled runs are **not guaranteed to run exactly on time**. They can be delayed by several minutes, or occasionally skipped, especially at busy times like the top of the hour.
> - Periodic pings **do not guarantee** that Render will keep a free service awake, and they do not override Render's platform policies or free-tier limits. This is a best-effort way to *reduce the chance* of idle spin-down. No uptime is guaranteed.

## Project structure

```
.
├── .github/
│   └── workflows/
│       └── keep-alive.yml
└── README.md
```

## 1. Create the GitHub repository

1. Sign in at <https://github.com> and click **+ → New repository**.
2. Name it (for example `render-keep-alive`).
3. Choose **Public**. See "Cost" below for why.
4. Click **Create repository**.
5. Upload the files (see "Uploading the files" below).

## 2. Add the `RENDER_BACKEND_URL` secret

1. In your repository, open **Settings → Secrets and variables → Actions**.
2. Click **New repository secret**.
3. Name: `RENDER_BACKEND_URL`
4. Secret: the full URL of your health-check endpoint, e.g. `https://YOUR-SERVICE.onrender.com/healthz`
5. Click **Add secret**.

The URL is never written in the workflow file, and GitHub masks secret values in logs.

## 3. Configure your health-check endpoint

- Use a **lightweight** route that returns HTTP `200` quickly and does no heavy work (no big database queries). A `/healthz` route is a common choice.
- The secret must contain the **full URL including `https://` and the path**.
- Test it first in your browser or with `curl -i https://YOUR-SERVICE.onrender.com/healthz`.

## 4. Enable, run, and monitor

**Enable:** Open the **Actions** tab. If GitHub shows a "workflows aren't being run" prompt, click **I understand my workflows, go ahead and enable them**. Scheduled workflows run from the **default branch** (usually `main`).

**Run manually:** Actions → **Keep Render Backend Alive** → **Run workflow** → choose the branch → **Run workflow**.

**Monitor:** In the Actions tab, open any run to see its log. A green check means the request succeeded; a red X means it failed. GitHub emails you about failed workflow runs by default (adjustable in your notification settings).

## 5. Troubleshooting

The log prints the HTTP status and curl's exit code (never the URL).

| Symptom | Likely cause | Fix |
|---|---|---|
| "Secret RENDER_BACKEND_URL is missing or empty" | Secret not set or misspelled | Re-add it with the exact name `RENDER_BACKEND_URL` |
| curl exit code **22** | Endpoint returned 4xx/5xx | Open the URL in a browser; check the path (e.g. `/healthz`) and your Render logs |
| curl exit code **28** | Timeout (60 s per attempt) | The service may be cold-starting or overloaded; re-run manually and check Render logs |
| curl exit code **6** or **7** | DNS or connection failure | Check the URL for typos and confirm the service is deployed and running |
| curl exit code **3** | Malformed URL | Make sure it starts with `https://` and has no spaces or quotes |
| Workflow never runs on schedule | Not enabled, wrong branch, or disabled by inactivity | Enable it in the Actions tab; the file must be on the default branch |

Other notes:
- **Delays are normal.** Runs may start several minutes late or be skipped.
- **60-day inactivity rule:** GitHub automatically disables scheduled workflows in public repositories after 60 days without repository activity. Re-enable it from the Actions tab, or make an occasional commit.
- A single failed run does not always mean a problem; the request retries up to 3 times before failing.

## Cost

- **Public repositories:** GitHub Actions is free and unmetered for standard runners.
- **Private repositories:** the free plan includes 2,000 minutes per month. Running every 10 minutes is about 4,300 runs per month, each billed as at least 1 minute, which would exceed that. Use a public repository to stay free.

Because the URL lives in a secret, making the repository public does not expose your backend address.
