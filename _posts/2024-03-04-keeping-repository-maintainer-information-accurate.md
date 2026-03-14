---
title: "Keeping Repository Maintainer Information Accurate"
date: 2024-03-04
categories: [Open Source, Tooling]
tags: [github-actions, codeowners, open-source, ospo]
image:
  path: https://github.blog/wp-content/uploads/2023/10/Collaboration-DarkMode-3.png?resize=1200%2C630
  alt: "GitHub collaboration illustration"
---

CODEOWNERS files are one of those things that start out accurate and slowly drift. People change teams, leave the company, or shift responsibilities - but the CODEOWNERS file stays frozen in time. The result? Review requests go to the wrong people, PRs sit waiting for approvals from folks who no longer work on that code, and ownership becomes murky.

To solve this, I built [cleanowners](https://github.com/github-community-projects/cleanowners) - a GitHub Action that automatically identifies stale or invalid entries in your CODEOWNERS files and opens pull requests to fix them. It checks whether the listed users and teams still exist, still have access to the repository, and flags entries that need attention.

**[Read the full article on the GitHub Blog →](https://github.blog/2024-03-04-keeping-repository-maintainer-information-accurate/)**
