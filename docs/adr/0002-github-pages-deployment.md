# ADR 0002: GitHub Pages Deployment Strategy

## Status
Accepted

## Context
We need to deploy our blog series on GitHub Pages while ensuring:
- Automated deployment process
- Version control integration
- Preview capabilities
- SEO optimization
- Performance optimization

## Decision
We will implement the following deployment strategy:

1. **Jekyll Configuration**
```yaml
# _config.yml
title: AAP on OpenShift VM Management
description: A comprehensive guide to managing VMs using Ansible Automation Platform on OpenShift
baseurl: "/aap4ocpv"
url: "https://your-org.github.io"

# Build settings
markdown: kramdown
permalink: /blog/:year/:month/:day/:title/
paginate: 5
paginate_path: /blog/page:num/

# Collections
collections:
  blog:
    output: true
    permalink: /blog/:title/

# Plugins
plugins:
  - jekyll-paginate
  - jekyll-seo-tag
  - jekyll-sitemap
  - jekyll-feed
```

2. **GitHub Actions Workflow**
```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          bundler-cache: true
          
      - name: Install dependencies
        run: |
          gem install bundler
          bundle install
          
      - name: Build site
        run: bundle exec jekyll build
        
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
```

3. **Branch Strategy**
- `main`: Production branch
- `develop`: Development branch
- `feature/*`: Feature branches
- `gh-pages`: Generated site

4. **Preview Process**
- Pull requests trigger preview builds
- Preview URLs available in PR comments
- Local development with `bundle exec jekyll serve`

## Consequences
### Positive
- Automated deployment process
- Version control integration
- Easy preview capabilities
- SEO optimization
- Fast page loads

### Negative
- Need to maintain GitHub Actions workflow
- Must handle dependency updates
- Requires careful branch management
- Need to monitor build times

## Implementation Plan
1. Set up Jekyll configuration
2. Create GitHub Actions workflow
3. Configure branch protection rules
4. Set up preview environment
5. Test deployment process 