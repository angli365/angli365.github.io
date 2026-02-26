# My Blogs

[![GitHub license](https://img.shields.io/github/license/cotes2020/chirpy-starter.svg?color=blue)][mit]

This is the source code for my personal technical blog, hosted at [https://icshare.work](https://icshare.work).

## About This Blog

A place for technical writing and personal reminders. Topics include software development, programming insights, and technical notes.

## Tech Stack

This blog is built with:
- **[Jekyll][jekyll]** - Static site generator
- **[Chirpy Theme][chirpy]** - Feature-rich Jekyll theme for technical writing
- **Ruby/Bundler** - Dependency management
- **GitHub Pages** - Hosting and continuous deployment

## Features

- 📝 Blog posts with categories and tags
- 🎨 Responsive design with dark/light theme toggle
- 🔍 SEO optimization
- 📱 PWA support for offline reading
- 📖 Automatic table of contents
- 🌐 Custom domain support

## Repository Structure

```
.
├── _posts/       # Published blog posts
├── _drafts/      # Draft posts (not published)
├── _tabs/        # Navigation pages (About, Archives, etc.)
├── _data/        # Data files and metadata
├── _plugins/     # Custom Jekyll plugins
├── assets/       # Images, CSS, JavaScript, and other static files
├── _config.yml   # Site configuration
└── index.html    # Home page
```

## Development

### Prerequisites

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Bundler](https://bundler.io/)
- [Git](https://git-scm.com/)

Follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of the basic environment.

### Local Setup

Clone the repository and install dependencies:

```console
$ git clone https://github.com/angli365/angli365.github.io.git
$ cd angli365.github.io
$ bundle install
```

### Running Locally

```console
$ bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

## Writing Posts

Create a new Markdown file in `_posts/` with the naming format:

```
YYYY-MM-DD-title.md
```

Use the following front matter template:

```yaml
---
title: Your Post Title
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [Category1, Category2]
tags: [tag1, tag2]
---
```

For more details, see the [Chirpy theme documentation][chirpy].

## License

This work is published under [MIT][mit] License.

[jekyll]: https://jekyllrb.com/
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE
