## Hype wave, pragmatic evidence vs the need to move fast

![](https://miro.medium.com/max/700/0*Ps0lgic20e019ANd.png)

workflow — all images by author

The Tech side of startups can sometimes be very fluid and contain a lot of unknowns. What tech stack to use? Which components might be overkill for now but worth keeping an eye on in the future? How to balance the pace of business features development while keeping the quality bar high enough to have a maintainable codebase?

Here I want to share our experience building [https://cleanbee.syzygy-ai.com/](https://cleanbee.syzygy-ai.com/) from the ground up — how we shaped our processes based on needs and how our processes evolved as we extended our tech stack with new components.

Businesses want to conquer the market and engineers — try cool stuff and stretch their brains. Meanwhile, the industry produces new languages, frameworks, and libraries in such quantities that you will not be able to check them all. And, usually, if you scratch the shiny surface of the Next Big Thing, you will find a good old concept. Good, if you are lucky.

One of the most exciting topics to argue about is the processes — whether you rely on [trunk-based development](https://trunkbaseddevelopment.com/), prefer a more monstrous [GitHub flow](https://www.endoflineblog.com/gitflow-considered-harmful), are a fan of [mobbing](https://www.agilealliance.org/glossary/mob-programming/), or find it more efficient to spend time in [PR-based](https://trunkbaseddevelopment.com/short-lived-feature-branches/) code reviews.

I have experience working in an environment where artifacts were thrown away on users without any standardized process. In case of issues, developers had a lot of fun (nope!) trying to figure out what version of components was actually deployed.

On another spectrum is the never-ending queue to CI. After you create PR you have to entertain yourself in the nearest 30 minutes by betting whether the CI cluster will find a resource to run tests over your changes. Sometimes the platform team introduces new, exciting, and useful features that might break compatibility with existing boilerplate for CI. These may result in failing all your checks at the last minute, after an hour of waiting.

I strongly believe that, as usual, it all depends on team maturity, the kind of software you are building, and various business constraints, for example, the existence of [error’s budget](https://about.gitlab.com/handbook/engineering/error-budgets/) and the importance of [time-to-market](https://enkonix.com/blog/time-to-market/) versus [SLXs](https://cloud.google.com/blog/products/devops-sre/sre-fundamentals-sli-vs-slo-vs-sla).

I think what is important is to have some agreed processes in place that everyone is aware of and follows. It’s also important to have the bravery to challenge and change it if there is evidence of a better alternative.

## Start Shaping the Process

What we have at the start:

-   less than a dozen developers — in-house team and temporary contractors who want to and [can work asynchronously](https://about.gitlab.com/company/culture/all-remote/asynchronous/)
-   completely greenfield project — no single line of code has been written yet. Requirements are vague, but they already started shaping into something
-   tech-wise — the clear need for a backend that should talk with mobile clients
-   some simple web frontend — static pages should be enough! (nope)

We have started simple — code at GitHub and a PR-based flow with a single requirement — to have tickets splittable to be delivered in 1–3 days. This required some practice of [story slicing](https://www.thoughtworks.com/en-de/insights/blog/slicing-your-development-work-multi-layer-cake), and it seems that a sense of visible fast progress is shown through the ability to move tickets to `Done`. This can be a great motivational factor for the team to onboard that idea.

Linters and static analyzers to skip exciting discussions, such as how many arguments per method are too much (6!). We’ll gradually add auto-tests. We also try [codesense](https://codescene.com/). They have a very promising approach to highlighting [important parts of code](https://codescene.io/docs/guides/technical/hotspots.html) (those bits that changed frequently and should definitely have a higher maintainability bar!) and identifying [complexity](https://codescene.com/blog/bumpy-road-code-complexity-in-context/) by looking at the level of nestness in the code. It is probably a bit expensive for startups initially, but 100% provides decent hints for engineers.

On the architecture side of things, there was a temptation to dive deep into the wonderland of microservices. But looking at [horrifying diagrams](https://www.divante.com/blog/10-companies-that-implemented-the-microservice-architecture-and-paved-the-way-for-others) of connections between them from big players, the need to [trace requests between](https://www.splunk.com/en_us/data-insider/what-is-distributed-tracing.html) them, it really seems a suicidal approach for teams in the early stage that want to move fast.

Analysis of requirements allow us to detect three groups of job:

-   core API with usual CRUD-like activities
-   search and recommendations
-   temporary workload that does something useful according to schedule (almost at a time with casual delays is OK)

![](https://miro.medium.com/max/700/0*d--YowWxj3b_d4r8.png)

Choice of tech stack: situations when the time is a bit limited, and expectations are high. [Use what you know and master](https://boringtechnology.club/) (yeah, maybe for someone, it is [boring technology](https://mcfunley.com/choose-boring-technology)). Hence, Fastapi, REST, stateless, Python, redis, and Postgres are our best friends (Yeah, we like Go and Rust, but we need to pay our dues a bit more!).

With mobile clients, the situation was a bit different. We foresaw a lot of screens with states and interactions with remote services but not too much custom, platform-specific tweaking. Hence, the idea of having a single codebase for iOS and Android was very appealing.

Nowadays, the choice of frameworks is really [wide](https://appinventiv.com/blog/cross-platform-app-frameworks/), but again, due to some experience with Flutter, we decided to give it a go. Within mobile development, one of the important aspects to better decide on is state management. Here, you will have a nice abundance of acronyms to be puzzled about from various languages and frameworks. Some include [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), [MVVM](https://medium.com/@buttonsrtoys/flutter-state-management-with-mvvm-1b55e6911975), [VIPER](https://www.kodeco.com/8440907-getting-started-with-the-viper-architecture-pattern), [TCA](https://www.kodeco.com/24550178-getting-started-with-the-composable-architecture), [RIBs](https://github.com/uber/RIBs/wiki), [BLOC](https://bloclibrary.dev/#/coreconcepts), [etc](https://docs.flutter.dev/development/data-and-backend/state-mgmt/options).

Our motto starts with the most simple (\*) solutions sufficient to support the necessary functionality. (\*) Simple. Well, let’s put it this way, we think we understand it.

However, we definitely made a mistake after building MVP because we decided to build on top instead of [throwing it away](https://errolhassall.com/blog/2019/11/2/why-you-need-to-throw-away-your-mvp). Hence, on one wonderful (nope!) sunny day, I questioned my sanity: after I commented out code, cleaned all possible caches, and still didn’t see my changes on a new screen. Yeah, the [dead code](https://wiki.sei.cmu.edu/confluence/display/c/MSC07-C.+Detect+and+remove+dead+code) should be removed!

## Start Building!

After those initial formalities were settled, the next necessary thing was to be able to check client-server interactions.

API [contract](https://en.wikipedia.org/wiki/Design_by_contract) is a great thing, but it will be much more obvious that something is wrong when a real server throws you a “schema validation error” or miserably fails with an HTTP 500 error code.

Backend services were initially split into two groups — API monolith and Search and Recommender. The first contains more or less straightforward logic to interact with DB, and the second contains CPU-intensive computations that might require specific hardware configuration. Every service has its own scalability group.

As we were still thinking about the rollout strategy (and arguing which domain to buy), the solution was simple: to minimize the struggles of mobile engineers in dealing with the backend, i.e., the alien stack. Let’s pack everything into docker.

When we prepare everything to be deployable locally — mobile engineers can run docker-compose commands and have everything ready (after a few painful attempts that reveal flaws in documentation, but the real value of such exercises is to react to every “WTF!” and improve it).

\`Everything\` is good, but what is the point of an API running on top of an empty DB? Manually entering necessary data shortly leads to depression (and the risk of increasing the duration of development cycles). Hence, we prepared a curated dataset that was inserted into the local DB to be able to play with. We also started using it for auto-tests. Win-win! Auth becomes less problematic in defining testing scenarios when you have dozens of dummy users with similar passwords!

## Try new things or choose third-party providers

Dealing with new technology is always a bit dangerous. You and your team can’t know everything (and sometimes things that you think you know can full you, but that’s another story). And still, it is often required to assess and investigate something no one has touched.

Payments, email, chat, SMS, notifications, analytics, etc. Every modern application usually represents business logics glued with several third-party providers.

Our approach to choosing with whom we work — time-capped, try-to-build-with-it activities to try the most promising one chosen by features, supported languages, and, in the case of providers, pricing.

## How did we get into Terraform?

The backend, a part of the DB, also should have some object/file storage. Sooner or later, we also should have DNS so that our services are ready to play with the big cruel world.

The choice of cloud provider was purely based on existing expertise within the team. We already use AWS for other projects, so we decided to stick with it. For sure, it is possible to do everything in the AWS console, but as times go, things become a classic big ball of mud that everyone is terrified to touch, and no one remembers why this bit exists at all.

OK, seems the paradigm of [infrastructure as code](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac) can be handy here.

Tooling-wise, choices are not so big — vendor-specific (AWS [Cloud formation](https://aws.amazon.com/ru/cloudformation/), Google Cloud ([Deployment Manager](https://cloud.google.com/deployment-manager/docs), [Azure Automation](https://learn.microsoft.com/en-us/azure/automation/overview)), [terraform](https://www.terraform.io/), and its [rivals](https://www.pulumi.com/).

Based on experience with terraform… you already got an idea of how we choose things?

Yeah, the initial setup will take some time (and without control can easily become the same big ball of mud in TF as well), but at least it will have some documentation over infrastructure and visibility of WHY it is there. Another major advantage, whatever you manage through TF, will be updated automatically (well, when you or CI/CD run corresponding commands)

## Secrets management

For AWS itself, given we run everything inside AWS, we can rely on [IAM](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) and [assumed roles](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) by attaching necessary policies to our VMs. But we need integration with third-party services and some way to pass some secrets to our apps, for example, passwords for DB. We need some solution for secret management. AWS has [KMS](https://aws.amazon.com/kms/), GitHub actions have [their own secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets), and apart from it, there are a bunch of other providers. So, the real question is: what do you need from secret management:

-   audit
-   path-based access
-   integration with Kubernetes
-   ability to issue temporary credentials
-   Web UI
-   Free
-   secrets versioning

KMS was very handy, and we managed to add it into GitHub actions, but the UI of [vault](https://www.vaultproject.io/) and the ability to use it for free (if you run it by yourself) was a kind of deal breaker on this matter.

## Path to Kubernetes

And once we have dockerized the app, we have started considering Kubernetes as it offers a few goodies out of the box. The most important one is to be able to spin up the necessary amount of pods to meet performance demands and the ability to define all your needs in a declarative fashion. So, given a sufficient level of automation, no human being should run `kubectl apply`. AWS has EKS to start with that can be managed via terraform.

On the other hand, a steep learning curve (to grasp the idea that it is exactly defined what should be up and running) and a bit of specific tooling to play with were fair reasons to think about it twice.

## Helm charts

If we talk Kubernetes and already have docker apps released on every merge to main, helm charts become the next step in adapting modern infrastructure stack. We have plugged AWS ECR to keep track of every new release and publish helm charts in dedicated S3 buckets that become our internal helm chart registry.

Plugging it all together was not as straightforward as expected. Kubernetes nodes initially can’t connect to ECR and pull necessary docker images, terraform modules (`aws-ssm-operator`) intended to work with secrets in AWS KMS were deprecated and didn’t support recent Kubernetes API, secrets, and config maps weren’t in the mood to be exposed into pods.

The first rollout of services brings happiness to mobile folks — no need to care about instructions for local setup! The initial week or so, though, it was not really stable, but then, one less thing to care about.

Do you need all of it? Not necessary.

I must admit, this mix — Kubernetes with vault via terraform and helm — are probably not for everyone, and you most likely will not need it in the initial stage. Simple docker push to ECR on merge to main and doing ssh into ec2 `&& docker pull && docker-compose` stop-start during release from CICD can work well (at least for a happy path). It will be clear for everyone at first glance. That’s exactly how we redeploy [our static websites](https://docs.aws.amazon.com/AmazonS3/latest/userguide/website-hosting-custom-domain-walkthrough.html) at the moment. We can focus on the ci build’s new version and copy it into the corresponding s3 bucket.

## Maturing the Infrastructure

AWS is nice enough to offer credits for those who are wild enough to explore the shady paths of the startup world. Can we use it to save a few bucks on GitHub minutes and expose fewer secrets and infrastructure to GitHub VMs?

How about self-hosted runners, i.e., when you open a PR, it is not GitHub VMs, but your own Kubernetes that’s allocated a pod to run your CI checks? Sure, it is difficult to prepare everything for iOS releases (more about it below), but Android and the backend should surely work on good old Linux?!

We have built it via dedicated [k8s pods](https://registry.terraform.io/modules/DNXLabs/eks-github-runner/aws/latest), but there is also an option to run checks on [on-spot EC2 instances](https://github.com/philips-labs/terraform-aws-github-runner).

## Observability and Co

There is a lot of marketing fluff around terms like monitoring and alerting.

In some companies, those things are implemented just for bragging, “We have X for that!”. However, engineers are still blind to what is happening with their production when there are real issues or alerts channels have to be muted as it contains nonactionable noise.

And I must say, we still have a looong way to go.

The first thing you will find as soon as you search for that kind of solution is [ELK stack](https://www.elastic.co/what-is/elk-stack) and a bunch of paid providers. After measuring the time and effort to maintain our own setup, I started thinking a paid solution might be worth it. If and only if you really can delegate the burden of squeezing the most important info about your apps and state of infrastructure to existing solutions. It all depends on whether they have a preset of metrics, log parsers, and index mapping that you can easily adapt to your project.

For logging, currently, we rely on [ELK](https://github.com/elastic/helm-charts). Yeah, it is more or less straightforward to set up, and most likely, some people find the query language of elastic very convenient to use daily.

Here we are still exploring options, as it seems that old good `kubectl logs` with `grep` produce insights for questions like “What is the last error from `app1` pods ” in a much more timely fashion without being lost among endless UI controls. But most probably, the UI of Kibana still hides the levers we should pull to add a proper ingestion pipeline and choose the corresponding mapping for the elastic index for filebeat.

For alerting, we set up [prometheus](https://github.com/prometheus-community/helm-charts) and integrate it into Slack. Again, mainly because we have experienced it before.

## Now, why do we need Azure?!

As usually happens when products evolve, new requirements introduce new kinds of things:

-   now, part of having something publicly visible, we need some resources available for the team only
-   to manage feature flags, access vault UI, or struggle with elastic to figure out the last API error

Sure, there are [paid solutions](https://www.cloudflare.com/products/zero-trust/) for that, or you can mix some `Identity` as a service provider (Azure active directory) for authentication of your teammates with any VPN providers. We chose [OpenVPN](https://openvpn.net/) due to their free tiers and expose necessary services to the internal network only so those who should log in using their credentials. It has one clear advantage compared to using AWS stack — it is free (for a limited number of connections).

## OK, why do we need Google Cloud?

So far, we have mainly discussed the backend part of things. But there are more. The thing that you see first — mobile apps! Flutter or something else must also be built, linted, and tested. And published somehow somewhere, so stakeholders can immediately be in awe of the new features (and find new bugs).

For rolling out to the production, you would need to pass through a bunch of formalities (screenshots, change log = what’s new, review) that will delay your audience from enjoying those art pieces.

I must say that the API of stores is not really friendly for frequent release. When you build and sign an app, publishing can take 15+ min. As with every other API, the API of app stores may and will fail sooner or later. Yes, and signing might be a nightmare as it differs between platforms. And it would be nice if engineers didn’t waste time on all of those things preparing releases from their laptops.

The first (and probably single?) thing that you should consider is [fastlane](https://fastlane.tools/) — initially, I did have some prejudice with all those new terms like [gems](https://en.wikipedia.org/wiki/RubyGems) (like that name, though!) and [bundle](https://bundler.io/), but it works. To run them from CI, some efforts will be required to deal with secrets [jks](https://developer.android.com/studio/publish/app-signing#sign_release) for Android or [match](https://docs.fastlane.tools/actions/match/) for iOS.

## Towards the “dark” side

Next, you will start thinking about app distribution: [testflight](https://developer.apple.com/testflight/) is a handy tool for the iOS world, but what about Android? We ended up using App Distribution — a solution from [Firebase](https://firebase.google.com/products/app-distribution) — mainly because it worked for us after the first try. But there are [other](https://testfairy.com/) options (that claim to be working for both platforms).

What is important is that you can do everything from fastlane! Even when your app evolves and you start adding various extras — analytics, chats, maps, geo — many were from Google directly of Firebase. As Firebase offers many goodies, it was natural to collect analytical events, and after a few tweaks with their IAM policy, we set up the export of raw events into gs-buckets to be able to play with BigQuery.

## Prod vs Staging — The Great Split!

![](https://miro.medium.com/max/700/0*vcK8iLuvJHnByyt4.jpg)

For backend, we have auto-tests right from the start. Various practices like test double prove quite efficient to prevent regressions even in complex business logic with integrations from side services. On the mobile side, we were a bit limited due to the coexistence of code from MVP, and auto-tests were not so helpful for complex business scenarios like someone wanting to use our services but we couldn’t charge their bank card.

Manual testing was very time-consuming and error-prone, especially when business logic dynamically evolved and the state of data in the database after recent updates became become impossible from the point of view of domain rules.

Yeah, so it would be nice to run e2e tests by clicking through the app with data that we are maintaining (and sure that it is valid). It would be nice if those tests didn’t pollute the actual database, S3 buckets, and third-party providers.

We started with a single main branch and a single environment (rds, redis, k8s namespace, and s3) used by the first testers and developers. We were not exposed to the public, but as we moved closer and closer to release, it became clear that some kind of distinction is necessary for places where we can break things and have a stable environment.

In mobile applications, changing the API’s URL during building was a matter. On the backend, a few aspects have to be done to support deploy-specific configurations: infrastructure-wise, by creating dedicated policies and resources and parameterized a few bits in the code where specific URLs were expected. Apart from it, there are several repositories, some of them independent but some are dependent — as in cases of shared functionality.

Do you know what will happen when you update shared functionality without immediate redeployment and testing all dependent apps? After a few days, when you completely forget about it, you make some innocent — purely cosmetic changes somewhere else in the dependent repo that will lead to redeployment and pull the latest dependency.

Surely, during an important dem or right after it, you would see some stupid errors related to a lack of compatibility for a single condition that you forgot to double-check.

1.  So, the first important consideration for splitting the environment — automate the overall rollout of all dependent applications if some base repo was updated. You may ask the team to do it, and everyone agrees but forget to run the pull.
2.  The second aspect — what do we need to deploy? Do we need to maintain all apps in every environment, including temporary jobs responsible for sending emails or notifications? Seems some flags to include or exclude jobs into deployment might be helpful.
3.  E2E, and later, probably Staging not necessary, should be reachable by everyone on the internet.
4.  Promoting new releases to e2e and staging have to be automated
5.  Promoting new releases to prod, at least now, better have controlled and manual

Currently, we have three ends, which fulfill all the things above:

-   E2E — an environment where integration tests will be run on curated data to ensure base functionality is still in place
-   Staging — where core development is happening and where beta testers can try to break what we build
-   Prod — that happy to greet new users

The Kubernetes cluster is still a single one. Everything was split on the namespace level. A similar thing happened with RDS, where several databases co-living together in an RDS instance.

![](https://miro.medium.com/max/700/0*cVf6AfJmOusTFMTa.png)

On the side of automation of mobile testing, the choice is not really big. You first have to choose whether you will use any device-in-the-cloud provider or run tests by yourself.

You can certainly plug a smartphone into a laptop and run tests, but wouldn’t it be nice (and right!) if CI did it instead? When you start considering vendors that provide emulators and real devices to play with, you will find the choice of testing framework for mobile is not wide, but that the second choice you have to make (and the choice of provider might limit you here). Another important consideration — is there specific hardware requirements, i.e., using GPU or npu? Hence, any emulator was sufficient for us.

We identify two main options for the mobile e2e testing framework — [Flutter integration tests](https://docs.flutter.dev/cookbook/testing/integration/introduction) and [appium-based pytests](https://appium.io/). Firebase Test Lab supported Flutter integration tests, although it required some tweaking to allow requests from their IP ranges (VM with running emulators) to reach our E2E API.

Appium, part of the Python API, was very promising, as it was like using something like a [testproject](https://testproject.io/) (you guys rock!). You can record all the clicks through the application per scenario. Hence, it doesn’t require specific programming knowledge, but it allows you to learn it gradually). So far, Appium is much more comprehensive in our setup regarding scenario coverage.

E2E tests have one tiny (nope!) issue — the cold start of the app in an emulator is not very fast. If we add the time necessary to build the app and the duration of copying the debug build to the provider on top of it, it becomes a real bottleneck of moving fast.

So far, we have experimented with running them twice in a day, but let’s see how it is going.

## What’s Next?

Many interesting tasks are still on our todo list:

-   On the infrastructure side — performance testing, security testing, trying out Flutter for web
-   On the development side — serving and updating ML models for recommendation engine, prediction of cleaning duration, building cache with feature vector for recommendations, intermixing [optimisation problems](https://en.wikipedia.org/wiki/Optimization_problem) to match the engine, and scheduling jobs and game theory

And most important, nothing can replace real-world usage.

You’ll see many wild things only when you start collecting real data about user behavior, so we are looking forward to the upcoming launch!