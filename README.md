# AI Batch 2024 blog: content

Posts and author profiles for the blog at <https://ai2024-chi.vercel.app/blog>.

This repository holds **content only**. The website lives in
[Shubhajeetgithub/FirstYearKGP](https://github.com/Shubhajeetgithub/FirstYearKGP); every time it
builds, it downloads the latest `main` of this repository. A push here checks the posts and then
triggers a rebuild, so a new post is live a couple of minutes after it is saved. Nothing in the
website repository changes.

## Writing a post

The easiest way is the online editor at <https://ai2024-chi.vercel.app/admin>. Sign in with
GitHub; you need **write access to this repository** (ask a maintainer). The full guide is at
<https://ai2024-chi.vercel.app/blog/contribute>.

You can also work with files directly:

```text
posts/
  my-post-slug/
    index.md        ← the post (copy TEMPLATE.md)
    figure.webp     ← images sit next to the post and are linked by file name
authors/
  firstname-lastname.json
```

- The folder name is the post's URL: `posts/my-post-slug/` becomes `/blog/my-post-slug`.
- Start from [TEMPLATE.md](TEMPLATE.md); it lists every frontmatter field.
- Maths: `$inline$` and `$$` display blocks, rendered with KaTeX. Nothing to switch on.
- Images: under 300 KB each, WebP or PNG. The site resizes and converts them.
- First post? Add yourself to `authors/` first. Your author id is the file name.

Open a pull request, or push to `main` if you have write access. The **Check and publish**
workflow builds the website with your changes and reports any problem (for example a missing
field or a broken image path) before anything goes live.

### Previewing locally

With both repositories cloned side by side:

```sh
cd FirstYearKGP
npm ci
BLOG_CONTENT_DIR=../FirstYearKGP-blog npm run dev   # PowerShell: $env:BLOG_CONTENT_DIR="../FirstYearKGP-blog"; npm run dev
```

Open <http://localhost:4321/blog>. Edits to posts reload automatically.

## Author profiles

`authors/<id>.json`:

```json
{
  "name": "Firstname Lastname",
  "github": "their-github-username",
  "role": "Contributor",
  "bio": "One line about you."
}
```

`avatar` (a photo URL) is optional and defaults to the GitHub avatar. `core: true` and `order`
are for the website's "Core Contributors" section.

## Maintainers

- **Publishing:** `.github/workflows/publish.yml` builds the website against this repository on
  every push and pull request, then, on `main`, calls the Vercel deploy hook stored in the
  repository secret `VERCEL_DEPLOY_HOOK`.
- **Access:** writers need Write access here only, not to the website repository.
