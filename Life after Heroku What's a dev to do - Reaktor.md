## Like most developers, we have our own toolbox of preferred platforms and apps that we turn to all the time. Heroku is one of those. 

Since its release in 2011, we’ve both been using it for all kinds of personal and work projects, and it’s been a lifesaver. Heroku lets you take a web application written in Node.js, Ruby on Rails, Python Flask, or pretty much anything else and with one click you can create a new Heroku app. From there, simply “git push” your application code to Heroku, and voila! You have a server on the web! It’s become a  trusted tool in our workflows. A big reason for that is that from the start, Heroku has had a really useful free tier along with an easy migration path to paid plans when you need more CPU, RAM, redundancy, and/or bandwidth. They even offer a very straightforward way to provision free PostgreSQL databases and much more.

###   
It was good while it lasted

So why are we talking about life _after_ Heroku? They’re not going away. At least, not exactly. What they are doing is [ending their free tier](https://blog.heroku.com/next-chapter) on November 28, 2022, creating a huge void for others to fill. Also, between us we have more than 25 applications running on free Heroku dynos ![😰](https://s.w.org/images/core/emoji/14.0.0/svg/1f630.svg). After talking with other developers, we’re definitely not the only ones who are trying to figure out what to do now that this change is fast approaching.

###   
Where to deploy now

With that in mind, we’ve personally investigated and analyzed what’s available, as well as asked friends at Reaktor what they would recommend as drop-in replacements. Here are our tried-and-tested suggestions for where you can deploy your small-scale hobby projects for free in the WAFH (World After Free Heroku):   

-   **[Fly.io](https://fly.io/)** lets you deploy Docker images and also supports setting up a PostgreSQL database at no charge. Their free tier includes 256MB RAM and also 3GB persistent storage, no sleeping dynos — which beats Heroku’s. You’ll most likely need a Dockerfile though, but it’s [not really that hard](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/), and learning Docker is a great transferable skill for your future projects. Plus, as a huge bonus, free tier is defined per organization, so you can have a separate free-tier quota for each of your projects! Their web-based tools as well as the CLI are nice to use.
-   **[Vercel](https://vercel.com/)** and **[Netlify](http://netlify.com/)** both allow you to deploy static sites and lambda functions, with Vercel especially tailored for deploying Next.js applications (React framework from the same company). In addition to a generous free tier, Vercel has a really nice user experience. As a cherry on the top, both automatically build a preview site for each Pull Request on Github. 
-   **[Render](https://render.com/render-vs-heroku-comparison)** has a [free tier](https://render.com/docs/free#free-web-services) similar to what Heroku has historically had: 512MB RAM; services spun down after 15 minutes of inactivity, implying a cold start delay; 750 hours max running time per month across all free Web Services in your account (there are roughly that many hours in a month); static sites are free. 
-   **[GitHub Pages](https://pages.github.com/)**  and **[Actions](https://github.blog/2022-08-10-github-pages-now-uses-actions-by-default/)** are also popular options for static sites. 
-   **[Heroku](https://www.heroku.com/)** could remain your go-to. At $7/month for hobby dynos, you can upgrade and keep your projects where they are and running with no interruption (except to your wallet). Higher tiers are of course also available. 

###   
And a database, plx

Many of our personal projects also rely on a database (PostgreSQL or Mongo), so we assume yours might as well. Heroku has been offering a free hosted PostgreSQL service with a limit of around 10k rows. So, what do the new kids offer?

-   **[Fly.io](https://fly.io/)** can run a max 1GB PostgreSQL database just like any other service and their CLI even offers to create a database when you create a new application. Very nice! The database volume (disk) is [backed up](https://fly.io/docs/rails/the-basics/backup-and-restoring-data/) on a daily basis and retained for 7 days, which is adequate for hobby projects.
-   **[Render](https://render.com/render-vs-heroku-comparison)** offers a PostgreSQL database for free for the first 90 days, with 1GB size limit and daily backups.

Additionally there’s a free tier on several database-as-service offerings that you can use. Here are a few tested options for popular databases.

-   **[ElephantSQL](http://elephantsql.com/)** and [Supabase](https://supabase.com/) offer hosted PostgreSQL databases  
-   **[PlanetScale](https://planetscale.com/)** offers MySQL compatible serverless databases
-   **[MongoDB Atlas](https://www.mongodb.com/atlas/database)** for your Mongo needs. (Juha’s been using their services for years now for [turtle-roy](https://turtle-roy.fly.dev/) without a glitch.)

###   
Observations & Trends

Since Heroku launched, the shape and structure of apps have changed quite a bit. Hosting options have also increased in variety, each bringing a different set of assumptions. During our discussions, we noticed a few trends emerging and wanted to list which category each option fell in:

-   **Static site builders which integrate Functions as a Service as a way to offer some compute.** These make use of services like AWS Lambda.
    -   Netlify
    -   Vercel
    -   Cloudflare Pages
-   **Container hosting services which require you to dockerise your app.** These options seem to have come around thanks to the development of container isolation projects such as gVisor and Firecracker.
    -   Fly.io
    -   Google Cloud Run
    -   Amazon’s AppRunner 
-   **Like Heroku, but different.** These are the closest to what might be called a drop-in replacement. 
    -   Render
    -   Railway
-   **Self-hosted open source alternatives.** With these, you can create “your own Heroku” for your team.
    -   Dokku

###   
Conclusion

As in other parts of life, sadly all good things must come to an end. (We just wish it weren’t so common with free dev tools!) Thankfully, there are some decent alternatives out there. Of course, the most suitable service depends on your exact needs.

So while our first reaction was to panic when we read Heroku’s announcement, we’re now kind of excited to see what these other platforms can do. Plus, we imagine this change will usher in a new wave of tools for simple app deployment. After all, necessity is the mother of invention, right? We’ll keep an eye on it, and if there’s anything that rolls out and is a great successor to Heroku, we’ll let you know — maybe in part 2!