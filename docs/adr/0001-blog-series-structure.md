# ADR 0001: Blog Series Structure and Organization

## Status
Accepted

## Context
We need to create a structured, maintainable blog series about Ansible Automation Platform (AAP) 2.5 on OpenShift VM management. The content needs to be organized in a way that:
- Builds knowledge progressively
- Maintains clear dependencies between posts
- Is easy to navigate and reference
- Can be deployed on GitHub Pages

## Decision
We will structure the blog series as follows:

1. **Directory Structure**
```
docs/
├── adr/                    # Architecture Decision Records
├── blog/                   # Blog posts
│   ├── part1-foundation.md
│   ├── part2-basic-ops.md
│   └── ...
├── images/                 # Images and diagrams
│   └── diagrams/
├── assets/                 # Additional assets
│   └── code-snippets/
└── _config.yml            # Jekyll configuration
```

2. **Post Structure**
Each blog post will follow this template:
```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
categories: [aap, openshift, virtualization]
tags: [ansible, automation, kubevirt]
series: "AAP on OpenShift VM Management"
part: X
previous: /blog/partX-1-title
next: /blog/partX+1-title
---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Implementation](#implementation)
4. [Testing](#testing)
5. [Troubleshooting](#troubleshooting)
6. [Next Steps](#next-steps)

[Content follows...]
```

3. **Cross-References**
- Each post will link to previous and next posts
- Technical terms will link to glossary
- Code snippets will link to full files
- Diagrams will be included with alt text

## Consequences
### Positive
- Clear, consistent structure
- Easy navigation between posts
- Maintainable content organization
- SEO-friendly structure
- Easy to extend with new content

### Negative
- Initial setup requires more effort
- Need to maintain consistent formatting
- Must ensure all links remain valid
- Requires careful version control

## Implementation Plan
1. Create directory structure
2. Set up Jekyll configuration
3. Create post templates
4. Implement navigation system
5. Set up GitHub Pages deployment 