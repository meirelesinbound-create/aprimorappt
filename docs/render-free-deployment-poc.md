# Render free-tier deployment proof of concept

## What this PR changes

Adds a root `render.yaml` Blueprint for a Docker-based Render Web Service using the existing repository Dockerfile. It does not change the app runtime, Cloudflare configuration, or production branch.

## How to try it without installing software

1. Open [Render](https://render.com/) and sign in with GitHub.
2. In the Render dashboard, choose **New + → Blueprint** and connect `meirelesinbound-create/aprimorappt`.
3. Select the branch containing this PR (`feat/render-free-deployment-poc`) while testing. Do not merge this PR just to run the test.
4. Review the Blueprint before applying it. It creates a Free web service named `aprimora-ppt`; no API-provider key is configured by this file.
5. Start the deployment and inspect the build/runtime logs. A failed build due to memory, time, or image constraints means this free host is not viable for the current full image.
6. If it starts, verify sign-in/authentication, generation, editing, PPTX export, and PDF export. Do not upload confidential client data during the test.

## Critical limitations

- Render Free services can spin down after inactivity and cold-start later.
- The filesystem is ephemeral. User configuration, local database state, uploads, and generated files may be lost on restart, redeploy, or instance replacement. Do not treat this as durable storage.
- The Docker image includes Python/FastAPI, Next.js, Chromium, OCR, and ML-related assets. Free-tier build/runtime resources may be insufficient.
- This Blueprint is not a guarantee that deployment will succeed or that generation will work.
- The service is not configured with a model-provider API key. AI generation will require the user to configure a supported provider securely through the application's settings or the host's secret environment variables, if appropriate.
- Do not connect a production domain or expose a service with unverified authentication, resource limits, or data persistence.

## Acceptance checklist

- [ ] Blueprint parses and service builds successfully.
- [ ] Service reaches a healthy state and homepage loads.
- [ ] Authentication is enforced before any user-specific data or exports are accessible.
- [ ] Generate a small test presentation.
- [ ] Edit a slide and export PPTX.
- [ ] Export PDF.
- [ ] Restart/redeploy and verify expected data behavior; document any loss.
- [ ] Confirm the actual free-tier limits and avoid any paid upgrade unless explicitly approved.

## Rollback

Delete the Render test service in the Render dashboard. The PR does not modify the existing Cloudflare deployment.
