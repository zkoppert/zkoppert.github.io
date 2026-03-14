---
title: "Self Building Jenkins CI Server"
date: 2020-08-03
categories: [DevOps, Automation]
tags: [jenkins, vagrant, ci-cd, infrastructure-as-code]
image:
  path: /assets/img/posts/vagrant.png
  alt: "Vagrant logo representing infrastructure as code for the self-building Jenkins project"
---

At a previous job we set up a Jenkins server to run some Continuous Integration (CI) on an InnerSource project that we were maintaining. No problem right?

Well... Who was going to maintain that server, and keep it secure, reproduce it if the server admin left? I decided that it would be more sustainable and reduce my workload to automate the building, configuration, and scaling of the Jenkins CI server.

## Build the CI server with one command

The goal was, as the heading states, to build/re-build the CI server with a single command: `vagrant up`. The company's tool of choice for automation was Jenkins so I wrote a Jenkins job to do just that. A general outline of the steps:

1. Build a Jenkins server using a Vagrantfile
2. Build several identical client servers using a Vagrantfile
3. Utilize the swarm-client project to have clients autoconnect to the Jenkins server
4. Codify build and test steps into the bootstrap files so they run as a part of `vagrant up`

Now every time that Jenkins crashed, a version upgrade was needed, or the hardware the server was being hosted on changed, we could just clone the repo and run `vagrant up`.

Time to recovery for this system was greatly improved by this automation. It went from approximately 24 hours to less than 1 hour.
