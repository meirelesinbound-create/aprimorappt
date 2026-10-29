# Zero-cost public hosting assessment

## Goal

Make Aprimora PPT available in a browser without requiring the owner to install software locally, while spending as close to zero as possible and retaining Cloudflare where practical.

## Finding: this repository is not a static site

The repository's root `Dockerfile` builds and runs the complete application: a Next.js frontend, a Python/FastAPI backend, Chromium, Node-based document/export tooling, OCR support, and supporting assets. The runtime starts the combined service with `node /app/start.js` and exposes port 80.

Therefore, a Cloudflare Pages deployment that runs `npx wrangler deploy` without a static output directory cannot deploy this application. Publishing only the frontend would leave generation, editing, and export API calls without their backend.

## Recommended approach

Treat Cloudflare as the public DNS/proxy layer if desired, but run the existing Docker application on a host that supports long-running containers. Do not add a static `dist` directory or a Worker shim as a substitute for the backend.

Before choosing a host, test the existing root Docker image and the actual generation/export workflow. The image includes Chromium, OCR and ML-related dependencies; its resource use and build time are likely to be much greater than those of a typical static website.

### Hosting options

| Option | Cost posture | Important constraints | Assessment |
|---|---|---|---|
| Cloudflare Workers / Pages Free | Free within published limits | Workers Free has a 10 ms CPU limit per invocation and does not provide Cloudflare Containers on the Free plan. The full app is a Docker service, not a static bundle. | Not suitable for hosting the complete app directly. Cloudflare can still handle DNS/proxying if another host runs the app. |
| Render Free web service using Docker | No monthly service charge for eligible free instances | Free services sleep after inactivity, have limited RAM/CPU, ephemeral filesystem, and are not intended for production. Large Docker builds and Presenton's browser/document-generation workload may exceed practical resource limits. Generated files and local database data must not be assumed durable. | Possible experiment only; validate build, memory, cold start, and export before relying on it. |
| Google Cloud Run free tier | Can fit within monthly free allowances for light use | Requires a billing account; usage beyond free allowances can incur charges. Container instances are ephemeral, so persistent artifacts and database state need external storage. | Technically plausible, but not a guaranteed zero-cost choice. Add budget alerts and billing safeguards before deploying. |
| Oracle Cloud Always Free VM | Potentially no compute charge when eligible capacity is available | Account/card verification, VM setup, Linux administration, firewall/security maintenance, and capacity availability add friction. It is not a managed one-click deployment. | Candidate if a stable always-on service is more important than simplicity and the account qualifies. |

Check the providers' current terms before deployment; free tiers and eligibility can change.

## Persistence and security requirements

Before exposing the application publicly:

1. Configure authentication. Do not expose an unauthenticated instance that lets arbitrary visitors consume AI-provider keys or generate files.
2. Store API keys and passwords only as host secrets/environment variables; never commit them.
3. Decide how to persist the application database, uploaded files, generated PPTX/PDF exports, templates, and user configuration. A container's writable layer is not durable storage.
4. Set usage limits and protect AI endpoints against abuse. The hosting free tier does not make third-party model APIs free.
5. Test a complete flow: sign in, submit a prompt/document, generate slides, edit a slide, export PPTX, export PDF, and verify files after a restart.
6. Add a budget alert or spending cap wherever the provider requires billing details.

## Suggested deployment sequence

1. Keep the current Cloudflare static deployment configuration unchanged until a supported runtime target is selected.
2. Test a Docker deployment of the existing root `Dockerfile` on a candidate host, without refactoring the application first.
3. Record build duration, image size, memory use, startup time, and end-to-end generation/export results.
4. If a free host cannot reliably run the workload, do not disguise that limitation with a frontend-only deployment. Either choose a provider with sufficient compute or split the product into a lightweight web UI plus a separately hosted generation service.
5. Once a host is chosen and its required secrets/storage are known, add a provider-specific deployment configuration in a separate implementation PR.

## Current conclusion

There is no verified, guaranteed-zero-cost, production-ready deployment target established by this repository alone. The lowest-risk next step is a bounded proof of concept on a free Docker-capable host, with Cloudflare retained as DNS/proxy where practical. This document intentionally does not add a provider-specific deployment file or claim that deployment has succeeded.
