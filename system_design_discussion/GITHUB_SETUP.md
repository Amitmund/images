# GitHub Setup for CDNUploadField

One-time setup per project (repeat this once for each Django deployment
that will use CDNUploadField — SREToolkit, kalaparichaya.com, etc.).

---

## Step 1 — Create a dedicated public repo for uploads

Do **not** reuse your application's source code repo. Keep uploads in a
separate, purpose-built repo — this keeps your actual codebase private
if you want it to be, while only the uploaded images themselves are
public (which is required for jsDelivr to serve them at all).

1. Go to https://github.com/new
2. Repository name: something like `sretoolkit-assets` or
   `kalaparichaya-assets`
3. **Visibility: Public** — this is not optional. jsDelivr's GitHub CDN
   only mirrors public repos; a private repo will not work no matter
   what token permissions you grant.
4. Check "Add a README file" (gives you an initial commit so the
   `main` branch exists before your app tries to push to it)
5. Create repository

## Step 2 — Generate a fine-grained Personal Access Token

Use a **fine-grained token scoped to only this one repo** — not a
classic token with blanket `repo` access. If it ever leaks, the damage
is contained to this one assets repo, not your entire GitHub account.

1. Go to https://github.com/settings/tokens?type=beta
2. Click "Generate new token"
3. **Token name**: something identifiable, e.g. `sretoolkit-cdn-upload`
4. **Expiration**: set a real expiration (90 days or 1 year) rather
   than "no expiration" — put a reminder on your calendar to rotate it
   before it lapses, since an expired token will start failing uploads
   silently until you notice
5. **Repository access**: "Only select repositories" → choose the
   assets repo from Step 1
6. **Permissions** → Repository permissions:
   - **Contents**: Read and write
   - **Metadata**: Read-only (this one is mandatory and auto-selected)
   - Leave everything else as "No access"
7. Click "Generate token"
8. **Copy the token immediately** — GitHub only shows it once. It looks
   like `github_pat_11AAAAAAA...`

## Step 3 — Add the credentials to your project's `.env`

```
CDN_UPLOAD_GITHUB_TOKEN=github_pat_11AAAAAAA...
CDN_UPLOAD_GITHUB_OWNER=your_github_username
CDN_UPLOAD_GITHUB_REPO=sretoolkit-assets
CDN_UPLOAD_GITHUB_BRANCH=main
```

Make sure `.env` is in `.gitignore` — this file must never be committed,
including to the private assets repo itself.

## Step 4 — Confirm settings.py is reading it

```python
from dotenv import load_dotenv
import os

load_dotenv(BASE_DIR / ".env")

CDN_UPLOAD_GITHUB_TOKEN  = os.environ["CDN_UPLOAD_GITHUB_TOKEN"]
CDN_UPLOAD_GITHUB_OWNER  = os.environ["CDN_UPLOAD_GITHUB_OWNER"]
CDN_UPLOAD_GITHUB_REPO   = os.environ["CDN_UPLOAD_GITHUB_REPO"]
CDN_UPLOAD_GITHUB_BRANCH = os.environ.get("CDN_UPLOAD_GITHUB_BRANCH", "main")
```

## Step 5 — Test it

Quickest sanity check — open `python manage.py shell` and run:

```python
from cdn_upload.storage import GitHubCDNStorage
from django.core.files.base import ContentFile

storage = GitHubCDNStorage()
url = storage.save("test/hello.txt", ContentFile(b"hello world"))
print(url)
```

You should get back something like:
```
https://cdn.jsdelivr.net/gh/your_username/sretoolkit-assets@main/test/hello-3f9a1b2c4d.txt
```

Open that URL in a browser — it should show "hello world". If it 404s
immediately, give it a minute; jsDelivr sometimes takes a short moment
to pick up a brand-new file for the first time (subsequent updates to
already-known files propagate faster).

If the `storage.save()` call itself raises an `IOError` instead, that
means the GitHub API call failed — double check:
- The token wasn't truncated/has no extra whitespace in `.env`
- The token's repo selection actually matches `CDN_UPLOAD_GITHUB_REPO`
- The branch name matches exactly (case-sensitive)

## Step 6 — Rotate the token before it expires

Since you set a real expiration in Step 2 (as you should have), put a
recurring reminder a week before that date. When it's time:

1. Generate a new fine-grained token the same way (Step 2)
2. Update `CDN_UPLOAD_GITHUB_TOKEN` in `.env` (and wherever your
   production secrets live — server env vars, deployment secrets, etc.)
3. Restart the Django process so it picks up the new value
4. Revoke the old token from https://github.com/settings/tokens?type=beta

## One repo per project, or one shared repo for everything?

Either works, but **one repo per project is the simpler default** and
what the settings above assume. If you'd rather have a single shared
assets repo across SREToolkit, kalaparichaya.com, and future sites, add
an extra top-level folder per project inside `upload_to`, e.g.:

```python
def event_image_upload_path(instance, filename):
    return f"sretoolkit/events/{instance.year.year}"
```

so everything from one project stays visually separated inside the
shared repo. Just be aware this means one leaked token, if it were a
classic token instead of a fine-grained one, would expose every
project's uploads at once — another reason to keep tokens fine-grained
and repo-scoped regardless of which layout you choose.
