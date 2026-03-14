# Launching an InnerSource Portal

**Project Description:** 
  In working as a consultant for companies, I have visibility into companies working hard to make InnerSource work.
  One crucial part of that is what's referred to as the "discovery" problem.
  The problem is essentially, once InnerSource projects start up as a company,
  it needs to be easy to discover those projects for anyone looking to start a new project that utilizes them.
  To solve this problem, in comes the InnerSource Project Portal.
  
## What is InnerSource
InnerSource is basically open source but inside a company or set of companies.
You can imagine that it might be desirable for a company to share reusable components internally,
but due to legal/monetization/other reasons they don't want to share beyond the company as open source.

## How to do it
SAP release a beautiful project called [SAP InnerSource Project Portal](https://github.com/SAP/project-portal-for-innersource).
As you can read from their `README.md`, the only pieces missing from this project is hosting, and an innersource crawler.
Luckily GitHub's recently announced [Pivate pages](https://pages.github.com/).
This makes hosting a one-click setup in the settings tab of the repository.
As for the innersource crawler, I have created [a project](https://github.com/zkoppert/innersource-crawler) to handle that part. Along with some GitHub actions magic, it all works great.
The GitHub action takes the repos.json created by the crawler and feeds it to the project portal website. Check out what this looks like below.

<img alt="InnerSource Project Portal" src="https://raw.githubusercontent.com/SAP/project-portal-for-innersource/main/docs/overview.png">
