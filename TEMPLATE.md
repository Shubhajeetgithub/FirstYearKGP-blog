---
# Copy this file to posts/<your-post-slug>/index.md. The folder name becomes the URL:
# posts/fourier-tricks/index.md is published at /blog/fourier-tricks.

# Required
title: Your post title
description: One or two sentences, 160 characters at most. Shown in search results and link previews.
pubDate: 2026-01-31              # YYYY-MM-DD
author: shubhajeet-das            # a file name in authors/ (without .json)
# author: [shubhajeet-das, kingshuk-patra]   # co-authored posts take a list
tags: [signals, exam-tips]       # lowercase; spaces become hyphens

# Optional
updatedDate: 2026-02-15
subject: AI20203                 # course code
cover: cover.webp                # an image next to this file; aim for ≤ 300 KB, WebP or PNG
coverAlt: Describe the cover image for screen readers
draft: true                      # drafts are checked but not published
---

Write the post here in Markdown.

## Headings

Use `##` and `###`. They become the table of contents.

## Maths

Inline: $e^{j\pi} + 1 = 0$. Display maths goes between `$$` lines:

$$
\int_{-\infty}^{\infty} e^{-x^2}\,dx = \sqrt{\pi}
$$

The site loads the maths styles automatically when a post contains maths.

## Images

Put the image next to this file and link it by name:

![Describe the image](my-figure.webp)

## Code

```python
print("hello")
```
