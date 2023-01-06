Being able to build products fast is a huge advantage at any stage of the company. As a founder, you always want to get things done to grow your startup and please your customers.

Over the past few years, I had an opportunity to work with many 0-4-year-old companies and I’ve been surprised that many of them have complex and slow software delivery processes. [Motivation](https://growing-products.paralect.com/two-essential-steps-to-build-high-performance-teams) is an essential part of being fast — but in this essay, we’re going to focus on the process and concrete steps to implement it.

One thing that I’d like to mention before we dive into the fast product development process is that if there is some third-party tool or service you can use to not build something — use it. Not building unless you absolutely need to is a great strategy.

TL;DR;

Go straight to the summary section at the bottom and copy the process.

### Avoid Feature Branches

Before diving into the details of the efficient process I’d like to mention one process which I often see companies using that usually slows down your delivery speed a lot. This process is known as [Feature Branches](https://martinfowler.com/bliki/FeatureBranch.html). Developers start a new branch to develop new a feature and only merge back when the feature is completed.

It often takes weeks or even months to build a single feature, which leads to the following problems:

-   merging the feature branch with the main branch takes a lot of time
-   when the feature is done, merging branches back might lead to new bugs and instability
-   it is more difficult to do incremental refactoring

This approach might work for smaller teams, but gets especially inefficient when the team grows to 5+ engineers.

If your team uses this approach your ability to deliver fast is at risk. Avoid it at all costs.

### Automate CI/CD pipeline

I hope every startup uses [Continues Integration](https://www.thoughtworks.com/continuous-integration) and [Continues Delivery](https://www.thoughtworks.com/insights/topic/continuous-delivery) — this is the automation of the build-test-release process. Or in simpler words, whenever a developer publishes their code an automated process should pick it up and publish it to the staging or production environment.

10 years ago [Jenkins](https://www.jenkins.io/) and [TeamCity](https://teamcity.com/gotime/) were the most popular options for CI/CD implementation. They required a lot of configuration, but worked well for process automation.

7 years ago tools like [Drone CI](https://www.drone.io/) and [Circle CI](https://circleci.com/) that used docker to build and test and test products became popular.

3 years ago [GitHub Actions](https://github.com/features/actions) become very popular. They also use docker images, but in addition tightly integrated with version control system and to allow creation of extremely complex CI/CD pipelines.

Over the last 2 years, very simple and integrated tools like [Vercel](https://vercel.com/) and [Render](https://render.com/) started to appear. These tools come with pre-built CI/CD pipelines which work out of the box and are very simple:

-   Developer pushes their changes to the main branch
-   Changes automatically propagated to the production environment

They also often automate routine DevOps tasks like environment variables management, SSL certificates, and basic scaling.

In fact, it does not matter what tools you use — the most important and first step towards faster delivery is fully automated CI/CD. That will save you a lot of time on manual deployments and will make it easy to release changes multiple times a day.

### Use a monorepo

A monorepo keeps things simple, organised and transparent — a central place where you can see what’s going on. Having multiple repos, especially once your team will start growing can lead to different approaches to building and deploying your product.

Key monorepo advantages are:

-   Visibility and single source for all code
-   Code sharing is much easier in a monorepo (and there are tools like [Turborepo](https://turborepo.org/), which make it even simpler)
-   It’s easier to find things in monorepo
-   Simple to organise CI/CD process, as all the code in one place

### Break features into small, incremental changes

Small, incremental changes are key to fast product development. With smaller changes, the team tends to apply better software development practices. Even small changes need to be tested to make sure nothing is broken and old functionality works as expected.

Here are three big reasons why incremental changes are important:

-   Implies faster feedback (this is especially important if the direction is wrong)
-   Fewer conflicts and easier to merge with changes of others
-   Reduces the probability of breaking big parts of the system

Small iterative changes increase the performance of your team. Normally, you can use two simple rules for engineers and product teams:

-   For the product team: any task should not take more than 3 days, 1-2 days on average.
-   For the engineering team: everyone on the engineering team should submit a pull request every two days, every day on average.

These rules cultivate iterative thinking for every member of your team. The product team needs to think and plan smaller iterations for features. The engineering team needs to merge their changes often and always think about how to merge even not completed features. Feature flags help solve this issue.

### Use feature flags

Feature flags (also known as feature toggles) is a technique which allows you to change product functionality without writing code. Engineers create feature flags and expose them in some way to the product and data teams.

Two key process changes feature flags bring which affect your team performance are:

1.  Engineers can publish partially done features, so changes are reviewed and merged often.
2.  The product team can review and provide feedback early on.

Here is a more in-depth review of [why feature flags are important](https://growing-products.paralect.com/5-reasons-why-top-product-teams-use-feature-flags-to-avoid-poor-ux). We created and use [Growthflags](https://growthflags.com/) to implement feature flags.

### Define owners

Defining a small team responsible for delivering each feature to production improves speed and quality. Usually, this team consists of someone responsible for the product and someone from the engineering team. Their goal is to deploy the feature — they’re not blocked by anyone else’s decisions and should resolve any blockers. They set a deadline and work to release that particular feature before this date.

Clearly defined ownership helps:

-   avoid the shared responsibility trap, when no one is responsible for something and work is not done.
-   trigger personal responsibility. The feature owner knows explicitly that no one else is going to complete their work.

Implementing owners could be as simple as creating a spreadsheet with Feature Name, Dev Owner, Product Owner & Release Date. In tools like Notion or Jira you can add custom columns for this purpose.

Use sprint/product planning to define feature ownership. Normally we would have monthly product planning, where we review last month's progress, set goals for the current, define owners and approximate deadlines for all features in the upcoming month.

During sprint planning, we usually spend most of the time breaking big features into the smallest possible deliverables.

### Request at least one demo per week

Once you’ve defined owners for every feature, you should define a clear process of recording demos. Demos help your team think like a user and discover issues, before your feature goes to production or even test environment.

If your team works in different time zones (which is often the case), offline demos give you the flexibility to provide feedback whenever you want. As a side effect, an early demo can be used as documentation for the QA, support and other product teams. Sharing early demos and getting feedback can dramatically improve product quality and delivery speed, as you reduce the number of iterations.

Typically I ask any team to record demos at the following stages:

1.  **Prototype demo:** The team builds an early preview of the feature, which is just UI in most cases and records a demo to share initial feedback, which often changes the direction of the feature.
2.  **Preview demo:** This is when the feature is somewhat functional, but not yet ready to be tested.
3.  **Beta demo:** The feature has already reached beta and your QA team is testing it. You can open and play around with the feature in the test environment.
4.  **Production demo:** The feature is tested and ready to be released to the first users.

The rule of thumb is that every team that works on the particular feature should record 1-2 demos of their work every week.

### Deploy weekly

With automated CI/CD pipelines deployment is as simple as merging changes from one branch into another. But, in fact, it always has associated risks:

-   database migrations can fail or work slowly
-   new features might not work as expected in production
-   app can become slower and affect user experience

The list could be continued, but the main idea is that even fully automated deployment should be watched by someone. The engineer who deploys the change should go and check if things are looking good at least a few times after release.

Making multiple deployments a week could be a good target for a bigger company. But for startups, it’s better to stick to weekly deployments at a predefined time.

That helps to:

-   Reduce risk of production failures, compared with daily deployments.
-   Save a lot of time, which the team can use to actually build new features.
-   Clearly define the cadence of your deployments which makes it easy to align marketing, support and other efforts.
-   Give some time to do manual QA.

With tools like Vercel and Render the deploy process itself could be as simple as:

-   Changes from the development branch are deployed to the test environment.
-   Changes from the main branch are deployed to the production environment.
-   The development branch is merged to the main once a week to do production deployment, and other changes are always merged into development.

### Freeze the code before release

This is not to improve the speed, but actually to keep quality more predictable. With our products, we have a manual QA engineer that verifies changes before they’re deployed to production.

Usually, the team stops merging big changes that could potentially affect a production release at least one day before deployment.

That gives some time for the QA engineer to review new changes and make sure they work as expected. Most of the time we would just hold merging new changes for a day.

### Write tests (or not?)

Tests are a great way to increase your ability to release more often and with a greater level of confidence. Increasing the amount of tests your team writes will have a positive impact on the quality of your software in the long run. That will help your team to switch from one deployment per week to two or even more.

Tests reduce pressure on the QA team, as they can focus more on testing edge cases and complex scenarios instead of testing basic key functions of your app like signin or signup. As you make big changes in the system, tests can help make them more safe and release new changes without breaking old features.

But, on another side tests slows you down as you build features. When your team writes tests they need 2x more time to release any feature. I believe that early-stage companies (0-2 years) that focus on product-market fit should write fewer tests.

At that stage companies often change direction and launch a lot of features to just experiment. So tests will reduce number of experiments you can launch. You may write tests for some key areas, which you believe won’t change, but you should certainly keep the number of tests low and focus more on building features for end users.

### Summary

So, to sum up the process that will increase your ability to ship features fast is this:

1.  Automate your CI/CD pipeline. Saves a lot of time on manual work and allows your team to release multiple times a day.
2.  Use a monorepo. That will keep things simple, organised and transparent within your startup.
3.  Break all features into small, incremental changes. That enables early feedback and reduces the chances of a big failure.

-   product team should break work into 1-2 days tasks
-   engineering team should push their changes daily

5.  Use feature flags.

-   engineering team won’t have fear to deploy
-   you can roll out features to a beta group of users
-   you can gather feedback from your team and external users faster
-   you can run a/b tests and perform experiments

7.  Define product and engineer owners for every feature.

-   to avoid shared responsibility and delivery delays
-   to encourage personal responsibility and unlock all talents of your team

9.  Request at least one demo per week. Motivate your team to share their work early, so you can discover bugs and provide feedback before too much time spent.

-   prototype → preview → beta → production — these are four demos you should ask your team to record for every feature
-   every feature team should record 1-2 demos every week

11.  Deploy weekly and follow simple CI/CD process.

-   use single branch (develop) for all development, deploy to test environment on every commit
-   merge develop branch into main once a week to deploy to production

13.  Freeze the code before release.

-   helps your QA team to test changes and ensure quality before production release.

15.  Don’t write many tests if you’re early stage startup.

-   tests are great in the long run, but can slow you down in the short run.

Written by Andrew Orsich. Thanks to Artem Kukharenko, Roman Loginov and John McTavish for reading drafts of this.