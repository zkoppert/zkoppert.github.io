---
title: "Launching an InnerSource Project Portal"
date: 2020-12-01
categories: [InnerSource, Tooling]
tags: [innersource, portal, github-pages, open-source]
image:
  path: https://raw.githubusercontent.com/SAP/project-portal-for-innersource/main/docs/overview.png
  alt: "InnerSource Project Portal showing a dashboard of discoverable internal projects"
---

In working as a consultant for companies, I have visibility into companies working hard to make InnerSource work. One crucial part of that is what's referred to as the "discovery" problem. The problem is essentially, once InnerSource projects start up at a company, it needs to be easy to discover those projects for anyone looking to start a new project that utilizes them. To solve this problem, in comes the InnerSource Project Portal.

## What is InnerSource

InnerSource is basically open source but inside a company or set of companies. You can imagine that it might be desirable for a company to share reusable components internally, but due to legal/monetization/other reasons they don't want to share beyond the company as open source.

## How to do it

SAP released a beautiful project called [SAP InnerSource Project Portal](https://github.com/SAP/project-portal-for-innersource). As you can read from their `README.md`, the only pieces missing from this project are hosting and an InnerSource crawler. Luckily GitHub's recently announced [Private Pages](https://pages.github.com/) makes hosting a one-click setup in the settings tab of the repository.

As for the InnerSource crawler, I have created [a project](https://github.com/zkoppert/innersource-crawler) to handle that part. Along with some GitHub Actions magic, it all works great. The GitHub Action takes the `repos.json` created by the crawler and feeds it to the project portal website.

![InnerSource Project Portal](https://raw.githubusercontent.com/SAP/project-portal-for-innersource/main/docs/overview.png){: .shadow }
_The SAP InnerSource Project Portal provides a discoverable catalog of internal projects._
