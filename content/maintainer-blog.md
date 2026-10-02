# From novice to maintainer - Reflections on a year of contributing to Knative

_(originally published on [IBM Developer](https://developer.ibm.com/))_

I made my first commit to the community Knative project in May 2021. Within a year, I had become a core maintainer. I looked back over that first year of contributing and came up with a list of 10 things that helped drive my success in the community.

## 1. Find a mentor

It can be hard to dive into open source, especially if it's something you're doing in your free time or alongside other commitments. Having someone to provide support and encouragement as you begin can be quite helpful. Ideally, this would be someone who has done some work in open source, but that's not an absolute requirement. In general, you want someone who can provide guidance and support, while also helping you set goals in your open source work.

## 2. Use the project

While it can be tempting to jump in and start writing code, it can also be helpful to start by approaching the project as an end user. Read the docs, set up a local deployment, and try some samples/tutorials. This will give you a good feel of how things are supposed to work and the types of things you can (and can't) do with the project.

## 3. Clone the repos and build from source

You'll be building from source frequently as an open source contributor, so you may as well spend some time early and get a process in place that works for you. Familarize yourself with the [general GitHub workflow](https://github.com/kubernetes/community/blob/master/contributors/guide/github-workflow.md): fork/clone > create branch > make changes > push changes > open pull request (PR). Figure out an easy way to build and run your updated code (for example, I wrote a little script to spin up and run on a local VM). As this is something you'll be doing over and over again, it's worthwhile to get something working early on in your open source career.

## 4. Read the docs

There are two benefits here: It'll help you learn about the project, and you'll come across a number of things that need fixing (broken links, outdated information, etc.) that you can open PRs for. This can be a great way to get started contributing to a project that's a little less technically demanding for users nervous about jumping into a new codebase.

## 5. Get to know the leads

These are the people who generally determine the evolution of the project -- what features to add, what code gets merged, what issues to address. You'll want to spend a little time getting to know these people. If there are weekly meetings for your project, try to attend and introduce yourself. If there's a Slack channel or discussion forum for the project, join it. Building a relationship with these people is key, especially if you want to start growing your responsibilities in a project.

## 6. Volunteer

If there's an open issue you want to work on, ask for it to be assigned to you. If there's something you think should be added to the project, open an issue for it. If there doesn't seem to be anything obvious to work on, ask if there are any areas that need help. Being proactive here is a good thing, and showing that you’re willing to pick something up will help you in the long run.

## 7. Review PRs

As new PRs come in, take a few minutes and read through them. This can help introduce you to the code base. The review comments can also be enlightening -- particularly as author(s) and reviewer(s) go back and forth about potential changes. This will also help you become familiar with code conventions and styles used by the project as well.

## 8. Triage issues

Similar to No. 7, take a look at issues opened against the project on a semi-regular basis. Maybe there's a question you can answer; you'd be surprised how many issues don't require expert knowledge to answer. Maybe there's a bug you can (or can't) reproduce; either way, that's valuable information to add to an issue. Maybe you notice the same issue popping up over and over again, and you might consider updating the docs.

## 9. Pay attention to the bigger stack

No project is an island. There will be things happening either upstream or in related areas that you might want to be aware of or introduce into your project. Staying abreast of them is a good way to provide value to the project and also pick up issues to work on.

## 10. Have fun

Contributing to open source is a great way to learn new technologies, meet new people, and build external eminence. Make sure to spend some time enjoying the experience!

## Next steps

To learn more about open source and Knative:

* Check out IBM's [commitment to open source](https://www.ibm.com/opensource).
* Explore Knative by trying the [Quickstart tutorial](https://knative.dev/docs/getting-started).
* See why serverless technologies like Knative are [the future of computing](https://developer.ibm.com/blogs/the-future-is-serverless).
* Set up your own Knative [development environment](https://developer.ibm.com/tutorials/develop-knative-serving-on-the-cloud) on IBM Cloud to get started contributing.
