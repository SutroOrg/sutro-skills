---
name: slang
description: Use when creating, editing, reviewing, or debugging SLang backends for Sutro, including entities, schemas, actions, triggers, security/auth, files, AI, HTTP calls, queues, cron, events, imports, and generated app behavior.
metadata:
  short-description: Work with Sutro SLang
---

# SLang

SLang is Sutro's backend definition language. Do not rely on this skill as a full language reference; use the official docs as the source of truth.

## Start Here

Read the relevant docs before writing or reviewing SLang:

- Overview: https://docs.withsutro.com/docs/SLang/introduction
- Data modeling, entities, schemas, fields, enums, relations: https://docs.withsutro.com/docs/SLang/data-modeling
- Actions, queries, mutations, control flow: https://docs.withsutro.com/docs/SLang/logic
- Auth, subjects, identity fields, groups, roles, permissions, `@subject.entity`: https://docs.withsutro.com/docs/SLang/security
- HTTP, queues, cron, events, trigger argument mapping, response shaping: https://docs.withsutro.com/docs/SLang/triggers
- File uploads, `FILE`, `store`, document storage: https://docs.withsutro.com/docs/SLang/files-and-storage
- Built-in modules, `AI.prompt`, `HTTP.fetch`, imports/exports: https://docs.withsutro.com/docs/SLang/modules
- Generated app security runtime behavior: https://docs.withsutro.com/docs/SLang/sutro-generated-app-security
- Quickstart, SAPI upload, and publishing flow: https://docs.withsutro.com/docs/getting-started/slang-quickstart
- Sutro API authentication and setup: https://docs.withsutro.com/docs/getting-started/api
- Upload and compile SLang through SAPI: https://docs.withsutro.com/api-reference/applications/update-an-application-through-slang
- Publish through SAPI: https://docs.withsutro.com/api-reference/applications/publish-an-application

## How to test, run, and deploy

To build a SLang app, users need to have a [Sutro](https://console.withsutro.com) account. To communicate with SAPI (Sutro's API), users also need to download their security bundle from [the console](https://console.withsutro.com/authentication).

SLang applications can be [uploaded to Sutro using SAPI](https://docs.withsutro.com/api-reference/applications/publish-an-application). During the upload process, SAPI will run a validation of the code and report back any eventual errors. To upload SLang code, you first need to create a project and application through SAPI.

If a SLang application is successfully uploaded to SAPI, it can be published (see the API docs for that), after which it will be available and can be integrated into other applications, e.g., a web frontend.

## Working Rules

- Keep SLang examples complete enough to validate through SAPI.
- Prefer documented syntax over older examples copied from tests or stale skill content.
- For auth, model a subject with `subject` and `identity <fieldName>`, use `@subject.entity` when action logic needs the persisted user row, and use documented `permissions Subject->relationPath->roleValue` syntax.
- Pretty much all apps need auth, so make sure there's at least one entity declared as `subject`
- For files, bind uploads from `@request.files.<fieldName>`, use `FILE` parameters/fields, and call `store` before persisting an upload that must survive the request.
- For AI and HTTP, import the built-ins with `import "AI"` or `import "HTTP"` and call their documented actions through the module namespace.
- For collection queries, remember `pageOf` returns `Page<Model>`; iterate over `page.items`.

## Validation

Validate SLang by uploading it through SAPI and publishing the application. Follow the SLang quickstart and SAPI reference links above, especially `PUT /applications/{applicationId}/slang` and `POST /applications/{applicationId}/publish`.
