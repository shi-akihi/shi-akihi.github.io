# Adding a post

Add a Markdown file to this directory, then add an entry to `blog/posts.json`:

```json
[
  {
    "file": "my-first-post.md",
    "title": "My First Post",
    "date": "2026-10-17",
    "excerpt": "A short description of the post."
  }
]
```

The Markdown file is rendered on its own page at `/blog/post.html?file=my-first-post.md`.
