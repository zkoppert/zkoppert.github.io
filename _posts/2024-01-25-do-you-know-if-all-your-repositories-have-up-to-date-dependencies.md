---
title: "Do You Know if All Your Repositories Have Up-to-Date Dependencies?"
date: 2024-01-25
categories: [Open Source, Tooling]
tags: [github-actions, dependencies, dependabot, open-source, ospo]
image:
  path: https://github.blog/wp-content/uploads/2023/12/Productivity-LightMode-2.png?resize=1200%2C630
  alt: "GitHub productivity illustration"
---

If you manage dozens or hundreds of repositories, it's surprisingly hard to answer a simple question: are all of them keeping their dependencies up to date? Some repos might have Dependabot enabled but PRs piling up unmerged. Others might not have it enabled at all.

The [evergreen](https://github.com/github-community-projects/evergreen) action scans your organization's repositories and identifies which ones are falling behind on dependency updates. It checks for Dependabot configuration, looks at whether dependency PRs are being merged, and generates a report showing which repos need attention.

**[Read the full article on the GitHub Blog →](https://github.blog/2024-01-25-do-you-know-if-all-your-repositories-have-up-to-date-dependencies/)**
