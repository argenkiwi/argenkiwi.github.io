# Binary Spells

> A blog about coding and software engineering.

Binary Spells is a personal technical blog focused on software architecture, design patterns, and modern development practices. This repository contains the source code for the site, which is built with [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/).

## 🚀 Getting Started

### Prerequisites

To run this site locally, you'll need:

- [Ruby](https://www.ruby-lang.org/en/downloads/) (3.0 or higher)
- [Bundler](https://bundler.io/)

### Local Development

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/argenkiwi/argenkiwi.github.io.git
    cd argenkiwi.github.io
    ```

2.  **Install dependencies:**
    ```bash
    bundle install
    ```

3.  **Start the Jekyll server:**
    ```bash
    bundle exec jekyll serve
    ```

4.  **View the site:**
    Open your browser and navigate to `http://localhost:4000`.

## 📂 Project Structure

- `_posts/`: Markdown files containing the blog articles.
- `_config.yml`: Global configuration for the Jekyll site.
- `index.md`: The home page of the blog.
- `about.md`: About page describing the blog's mission.
- `Gemfile`: Ruby dependencies, including the `github-pages` gem.

## ✍️ Writing New Posts

To add a new post, create a file in the `_posts/` directory following the naming convention `YYYY-MM-DD-title.md`. Ensure the file starts with the required YAML front matter:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +0000
tags: [Tag1, Tag2]
---
```

> [!TIP]
> Use the `github-pages` gem to ensure your local environment matches the GitHub Pages production environment exactly.
