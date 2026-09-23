---
title: Docusaurus Blog Setup
sidebar_position: 1
---

# Docusaurus Blog Setup

A personalization journey of the Developer Akademie Docusaurus template 
for the DevSecOps portfolio project, covering configuration, customization 
and lessons learned along the way.

## TOC

- [Quickstart](#quickstart)
- [Description](#description)
- [Further References](#further-references)

import GithubLinkAdmonition from '@site/src/components/GithubLinkAdmonition';

<GithubLinkAdmonition 
    link="https://github.com/ralfs-devs/dev-blog-template"
    title="GitHub Repository" 
    type="tip"
>
Checkout this repository to see the full configuration 
and implementation details
</GithubLinkAdmonition>

## Quickstart

1. Clone the repository locally
2. Install dependencies: `$ npm install`
3. Configure environment variables in `.env` (copy from `example.env`)
4. Start local development server: `$ npm start`
5. Build for deployment: `$ npm run build`

## Description

This project started from the Developer Akademie Docusaurus template 
and was customized for personal portfolio use. 
Below is a summary of the key configuration changes made during the setup process.

### Environment Variables

Created `GIT_REPOSITORY_URL` in `example.env` 
and added corresponding TypeScript variable `gitRepositoryUrl` 
following the `blogEnabled` pattern. 
This centralized all repository URLs throughout the configuration.

### Configuration Changes in `docusaurus.config.ts`

- Personalized `title` and `tagline`
- Updated `url`, `baseUrl`, `organizationName`, and `projectName` defaults
- Replaced hardcoded GitHub URLs with `gitRepositoryUrl` variable
- Updated Navbar (title, logo alt text, GitHub href)
- Restructured Footer:
  - Added Projects link to Docs column
  - Removed Community section entirely
  - Customized More column with GitHub and Template references
  - Updated copyright notice with attribution

### Breaking Challenges Encountered

**1. Broken Links on Section Routes**

Adding a `/docs/projects` link to the footer caused the build to fail 
because the route didn't exist. Docusaurus's `onBrokenLinks: 'throw'` 
setting prevented deployment until all links were valid.

**Solution:** Created `docs/projects/index.md` 
as the landing page for the section.

**2. Fragile Footer Indexing**

Removing the Community section shifted all remaining footer sections 
by one index position. 
Code referencing `links[2]` broke silently until a build was triggered.

**Solution:** Switched from index-based access (`links[1]`) 
to title-based lookup (`find(column => column.title === 'More')`). 
This makes the code resilient to future footer layout changes.

**3. TypeScript Type Warnings**

Docusaurus's loose type definitions for `themeConfig` triggered warnings 
when accessing dynamic properties like `navbar.items` and `footer.links`.

**Solution:** Replaced `as any` casts with local type definitions 
and optional chaining. While more verbose, 
this keeps TypeScript validation active and catches typos at compile time.

```typescript
const navbar = config.themeConfig?.navbar as
  { items?: Array<{ to?: string; label?: string; position?: string }> }
  | undefined;

const footer = config.themeConfig?.footer as
  { links?: Array<{ title?: string; items?: Array<{ to: string; label: string }> }> }
  | undefined;
```

## Further References

- [Docusaurus Documentation](https://docusaurus.io/docs) 
— Official framework guides
- [Developer Akademie Template]
(https://github.com/Developer-Akademie-DevSecOpsKurs/dev-blog-template) 
— Original starter template
- [GitHub Pages Deployment](https://docs.github.com/en/pages) 
— Hosting on GitHub Pages
- [TypeScript Handbook](https://www.typescriptlang.org/docs/) 
— Advanced type definitions and optional chaining  