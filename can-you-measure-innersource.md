<img alt="Collabocats - two GitHub Octocats working together at a shared desk, representing cross-team collaboration" src="https://octodex.github.com/images/collabocats.jpg"/>

# Can You Measure InnerSource?

I've been asked this question more times than I can count. It usually comes up right after someone pitches an InnerSource program, gets conditional approval, and then gets told to prove it's working.

I know this because it happened to me. At a previous company, I was able to convey the need for InnerSource by demonstrating how much rework we were doing across departments. The CEO didn't like hearing about rework and waste, so we got approval - with a catch. The condition was: "Figure out how to measure if it's working." I walked out of that meeting excited about the green light but also terrified, because I had no idea how to actually measure InnerSource success.

This story isn't unique. I've heard from many others in the InnerSource community who face the same challenge. And I've also seen InnerSource programs get cancelled because stakeholders couldn't see tangible value. When you can't demonstrate return on investment, programs lose funding and support. That's not just a measurement problem - that becomes a sustainability problem.

So I built a tool to help. And in this post, I want to share the framework I've developed for measuring InnerSource, the common mistakes I see organizations make, and how you can start measuring today.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ao9JLzX7yK4" title="Can You Measure InnerSource? - Conference Talk" style="border:0;" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

*This post is based on my conference talk at the InnerSource Commons Summit. You can watch the full recording above.*

## Why measuring InnerSource is hard

Let me start with a confession: measuring InnerSource is genuinely difficult. There are three core reasons why.

**The Complexity Problem.** InnerSource isn't just about code. It's about culture, collaboration, knowledge sharing, and business outcomes. Unlike traditional software metrics where you might measure uptime or error rates, InnerSource success spans multiple dimensions that are inherently interwoven together.

**The Timing Problem.** Measurement needs to happen with awareness of team structures at the point in time when a contribution is made. Two people on the same team working on their own roadmap isn't InnerSource. Two people from different teams collaborating on a project both teams need is. And since people change teams over time, measuring InnerSource activity requires context about who was on which team when the contribution happened.

**The Definition Problem.** In order to measure InnerSource, we have to define the relevant terms. What counts as an InnerSource contribution - code, documentation, reviews, ideas? What is considered an InnerSource project - a codebase, library, platform, or tool? And what InnerSource contribution rates would you like to see - 5% or 50%?

## Three core measurements

I built the [`measure-innersource`](https://github.com/github-community-projects/measure-innersource) tool to solve this problem. Through using it across multiple organizations, I've learned that successful InnerSource measurement comes down to three core areas.

### 1. Cross-team participation

This answers the fundamental question: "Is InnerSource actually happening?"

The tool automatically identifies all contributors to your repository, maps them to teams using your organizational structure, categorizes contributions as either from the owning team or from other teams, and calculates your **InnerSource ratio** - the percentage of work done by external teams.

In one report I generated, we discovered that while a repository had 25 total contributors, only 8 were from the owning team. That's a 68% InnerSource participation rate. The tool showed us exactly who these external contributors were and which teams they came from.

If you want to dig deeper into what different InnerSource percentages actually mean in practice - from 0% all the way to 50% - I covered that in detail in an [earlier post on measuring innersource across your organization](https://github.blog/2022-05-16-how-to-measure-innersource-across-your-organization/).

### 2. Contribution volume

The tool gives you a complete picture of InnerSource activity by counting commits, pull requests, and issues - broken down by individual contributor.

This granular data helps you understand the quality and depth of cross-team collaboration. Are your InnerSource contributors making substantial code contributions, or are they primarily filing issues? Are certain external teams contributing more than others?

For example, one repository showed 23 total InnerSource contributions - 15 commits, 5 pull requests, and 3 issues - coming from 4 different teams. That tells a valuable story about meaningful technical collaboration, not just casual engagement.

### 3. Cross-repository impact

Here's where the tool becomes really powerful for demonstrating business value. Run it across multiple repositories to identify:

- **Which projects attract the most InnerSource collaboration** - these are your high-value shared assets
- **Which teams are the most active InnerSource contributors** - these are your collaboration champions
- **Which repositories have low InnerSource ratios** - these might be opportunities for improvement, or they might just serve one product and that's okay too

This all leads to your ROI story. If you run the tool on 20 repositories and find an average 35% InnerSource ratio, that means 35% of your development work is being shared across team boundaries. That's reduced duplication, shared maintenance costs, and faster innovation through collaboration. Now imagine taking that number down to zero - you'd be duplicating effort and wasting resources.

## What not to do

Before you start measuring, let me save you from the mistakes I see organizations make all the time.

**Mistake #1: Git-itis.** "We have 10,000 commits this quarter (up 20%!) - InnerSource is working!" Git activity is a lagging indicator at best, and often misleading. High commit volume might indicate inefficient development practices, not healthy collaboration.

**Mistake #2: Vanity metrics obsession.** Measuring things like "number of repositories" or "total contributors" without context. It's like measuring a company's success by counting how many meetings they have. The metrics need to tie back to meaningful outcomes.

**Mistake #3: The measurement death spiral.** Over-measuring and under-acting. I've seen organizations collect 47 different InnerSource metrics monthly but never use them to make decisions.

Here's my litmus test: After collecting your metrics, can you answer this question in one sentence - "Based on our data, what should we do differently next month?" If you can't, you're measuring the wrong things.

## Get started in 30 minutes

The [measure-innersource](https://github.com/github-community-projects/measure-innersource) tool is completely free and open source. It runs as a GitHub Action - no infrastructure needed. It automatically calculates InnerSource ratios and generates comprehensive markdown reports.

What you need to get started:

1. An `org-data.json` file mapping your team structure
2. A GitHub repository with the workflow file
3. A GitHub token for API access

Here's how to start measuring InnerSource in the next 30 minutes:

**Minutes 1-10**: Create your `org-data.json` file with your team structure

**Minutes 11-20**: Copy the GitHub Action workflow to `.github/workflows/measure-innersource.yml`

**Minutes 21-30**: Commit, push, and watch your first InnerSource report generate automatically

Check out the [README](https://github.com/github-community-projects/measure-innersource#readme) for the full setup guide and example workflow configuration.

## Your challenge

I want to leave you with two questions and a challenge.

Would measuring InnerSource in this way help you convey the value of your InnerSource program? Would it help you make better decisions to improve it?

If the answer to either of those is "yes," I challenge you to find time on your calendar this week to take a look at the tool and try it out. It has helped me and the organizations I've worked with, and I'd love for it to help you too.

The problem of measuring InnerSource is genuinely hard. It requires defining terms, understanding context, and focusing on meaningful outcomes. But it's absolutely worth the effort to demonstrate and increase the value of InnerSource in any organization.

InnerSource measurement isn't about agreeing on ideals - it's about demonstrating the value of the practice.

If you have questions or want to share your own InnerSource measurement journey, feel free to [open an issue](https://github.com/github-community-projects/measure-innersource/issues) on the project. I'd love to hear from you.
