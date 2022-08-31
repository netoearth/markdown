If you've been thinking about creating a dev blog for yourself, you've probably been a bit overwhelmed by the number of tools and technologies. We live in an era of abundance, and there are _a lot_ of options.

When I was building this blog, my biggest priority was to find a solution that would let me embed _totally custom content_ in each post, like this exploding-logo animation thing. When using markdown or a rich-text editor in a CMS, it's not at all clear how to do this: you're generally limited to the handful of HTML elements that these tools can render to.

In this article, I'm going to break down how my blog works, so that you can build something similar for yourself. I'll also cover all the most-commonly-asked questions I've gotten over the years. It's not a tutorial, but it should give you a broad roadmap to follow.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#the-stack)The stack

This blog is a [Next.js](https://nextjs.org/) application.

With Next, you have a few different options when it comes to page rendering: you can choose to do it “on-demand” (server-side rendering) or ahead of time (static site generation). I've opted to build all the blog posts ahead of time, when the site is generated.

I also use Next's [API Routes](https://nextjs.org/docs/api-routes/introduction) for things that require persistence in the backend. I use [MongoDB](https://www.mongodb.com/) as my database, to store stuff like the # of likes each post has.

I deploy this blog on [Vercel](https://vercel.com/). I initially chose them because they're the company behind Next.js, and I figured it would be well-optimized. Honestly, their platform is awesome. I wound up moving some of my non-Next projects there as well.

When it comes to the styling, I use [styled-components](http://styled-components.com/), and write all the styles from scratch. I don't use any "cosmetic" libraries like Bootstrap (and [I don't think you should either](https://www.joshwcomeau.com/newsletter-issues/012/#thoughts-on-css-frameworks-and-stylized-component-libraries)). I do use [Reach UI](https://reach.tech/) for things like modals, though.

For animation, I mainly rely on [React Spring](https://www.react-spring.io/), though I've started dabbling with [Framer Motion](https://www.framer.com/motion/) recently.

But the most critical part of my stack is [MDX](https://mdxjs.com/).

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#mdx-the-secret-ingredient)MDX, The secret ingredient

MDX is an extension of Markdown that allows you to import and use custom React components.

Even if you've never written Markdown, you've probably seen it before. It's a widely-used format — all those `README.md` files that are shown on Github repositories are Markdown!

Here's what Markdown looks like:

When using Markdown in a web application, there's a "compile" step; the Markdown needs to be transformed into HTML, so that it can be understood by the browser. Those asterisks get turned into a `<strong>` tag, the list gets turned into a `<ul>`, and each paragraph gets a `<p>` tag.

This is great, but it means we're limited to a handful of HTML elements that Markdown is aware of.

MDX takes the format a step further, and allows us to include _our own_ elements, in the form of React components:

We can create our own rich set of primitives, and use them in our content. On this blog, for example, I'm not limited to _italic_ and **bold** text; I also have _spicy text_ and .

We can also create custom one-off widgets. In [“A Friendly Introduction to Spring Physics”](https://www.joshwcomeau.com/animation/a-friendly-introduction-to-spring-physics/), an article all about motion and animation, I wanted readers to be able to play and experiment with the physics. So I created a bespoke React component, `SpringMechanism`:

(Drag and release the weight! Alternatively, you can also focus it and press "Space".)

When it came to explaining technical elements, like the effects of tension or mass on the physics, I added props to this component, to change how it behaves. This side-by-side demo shows how _mass_ affects the animation:

This is _so much more powerful_ than describing the physics in words, or showing the effects with a video. By giving the reader control, we flip from passive learning to active learning.

If you're a React developer, hopefully this is giving you a ton of ideas. Literally anything you do in a React app can now be embedded anywhere inside your blog posts!

You might be wondering: why not create a "regular" React application, and render each post as its own route? Why bother with MDX at all?

When I first started my blog, this is exactly what I did. It worked OK, but MDX is better for two reasons:

1.  The authoring experience is _so much nicer_. I don't need to wrap each paragraph in a `<p>` tag. I can use asterisks and underscores for strong/em tags. It's delightful.
    
2.  Even more importantly, _Markdown is data_. I can extract metadata from each post, so that I can show a filtered, sorted list of posts on the homepage, or all the posts about CSS on the [CSS category page](https://www.joshwcomeau.com/tutorials/css/). I wouldn't be able to do this as easily if each post was its own React component.
    

In my mind, MDX is the perfect sweet spot between “all-code” (a standard React application) and “all-data” (formatted text stored in a CMS).

It's not the only way to accomplish this, but it's the lowest-friction way I've found for developers working on a solo blog.

In the future, I plan on writing an “Intro to MDX” blog post. For now, though, I'll defer to these amazing community resources:

-   [“The X in MDX”](https://www.youtube.com/watch?v=xEu3t-KJVVg), an eye-opening talk from Rodrigo Pombo
    
-   [MDX v2 Syntax](https://egghead.io/lessons/egghead-mdx-v2-syntax), a wonderful video lesson by Laurie Barth
    

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#using-mdx-with-nextjs)Using MDX with Next.js

As I write this, there are four (4) different popular ways to use MDX with Next.js 😅

There's:

1.  The official way, with [@next/mdx](https://www.npmjs.com/package/@next/mdx)
    

4.  Kent C Dodds' [mdx-bundler](https://github.com/kentcdodds/mdx-bundler)
    

On this blog, I use next-mdx-enhanced, but I mainly chose it because I was migrating this blog from Gatsby to Next, and it was the option with the least amount of friction.

On my [course platform](https://www.joshwcomeau.com/css/the-importance-of-learning-css/#css-for-javascript-developers), I use next-mdx-remote. Version 3.0 just shipped, and it improves/fixes a bunch of stuff.

The biggest remaining drawback is that you can't import one-off components inside MDX files. Every bespoke component you create must be packed together inside one monolithic MDX bundle. In order to avoid an enormous bundle filesize, I need to make heavy use of lazy-loading, which adds an annoying bit of friction to the developer experience, and worsens the user experience.

Going forward, I'll probaby use mdx-bundler. It offers many of the benefits of next-mdx-remote, but without the "no import" drawback.

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#metadata)Metadata

In addition to the content itself, we need a way to store "metadata" — stuff like the title, the abstract, the publication date.

On my blog, I use _frontmatter_. Frontmatter is a Markdown addon that lets us define key-value pairs at the top of the document.

Here's what this post looks like:

The mechanism for defining and accessing metadata will vary depending on your MDX tool. In my case, `layout` refers to the React page component that will be used (`Article`). When this post is rendered, the `Article` component will be passed two props: `frontMatter` and `children`:

In this example, I'll wind up with an h1 that reads “How I Built my Blog”, followed by the article content.

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#index-pages)Index pages

On the [homepage](https://www.joshwcomeau.com/) of this blog, I have two separate lists of posts:

1.  The 20 latest posts, in chronological order
    
2.  The 10 most-viewed posts of all time
    

![A screenshot of the homepage, with relevant areas labeled: a list of new posts, and a list of popular ones](https://www.joshwcomeau.com/_next/image/?url=%2Fimages%2Fhow-i-built-my-blog%2Fhomepage-areas.png&w=3840&q=75)

Using the method `getStaticProps`, Next allows us to do some work when the site is built, before it gets deployed. I calculate the lists of posts to be displayed in these sections during that time.

Here's what that looks like:

`getLatestContent` is a method that traverses the local filesystem to find all of the `.mdx` blog posts. The logic looks something like this:

1.  Collect all of the MDX files in the `pages` directory, using `fs.readdirSync`.
    
2.  Load the frontmatter (I use an NPM package for this, [gray-matter](http://npmjs.com/package/gray-matter)).
    
3.  Filter out any unpublished posts (ones where `isPublished` is not set to `true`).
    
4.  Sort all of the blog posts by `publishedOn`, and slice out everything after the specified `limit`.
    
5.  Return the data.
    

This feels surprisingly low-level, especially when coming from Gatsby (where all this data was available magically through GraphQL). Ultimately, though, I kinda like it. It's definitely more work, but it gives me a ton of control.

This control comes in handy when it comes to the other method listed here, `getPopularContent`. It's very similar to `getLatestContent`, but it makes a database request (I rely on the hit-counter data stored in MongoDB to determine popularity).

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#api-routes)API routes

This blog is mostly a static site, but there are some data-driven aspects. For example, each article comes with a 90s-inspired hit counter!

I've previously written about [how I built my hit counter](https://www.joshwcomeau.com/gatsby/using-netlify-functions-with-gatsby/), using Netlify Functions and Gatsby. The process is similar with Next and Vercel API routes. You can learn more about it [in the official docs](https://nextjs.org/docs/api-routes/introduction).

I use a similar mechanism for the "Like" counter, the cute heart-shaped button that lets readers inflate my ego.

Each user is allowed to “like” each post up to 16 times. Initially, I tracked this in localStorage, but one of my favourite people on Twitter showed me why this is a bad idea by [adding almost 40k fake likes to a post](https://twitter.com/wongmjane/status/1232325459842482176).

Fortunately, Vercel includes the user's IP address in the request headers. When a user clicks the heart, I hash their IP address (to protect privacy) and check to make sure they haven't already hit the limit.

In my database, I have a big map for each post, tracking likes by user:

It's hard to overstate how powerful API routes are. This blog is a static site with relatively modest backend needs, but I'm using the same stack on my course platform, a full-fledged dynamic web application, with user authentication and roles and transactional email and all sorts of stuff. It holds up _super well_.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#build-helpers)Build helpers

As I've mentioned, I migrated my site from Gatsby to Next.js. I did this primarily because I was sick of context-switching: I built a course platform using Next, and wanted both projects to use the same stack.

Because Next is relatively unopinionated when it comes to app structure and data, there isn't the same rich ecosystem of plugins. Before, I had been generating an RSS feed and a sitemap from a handy-dandy Gatsby plugin. As part of the migration, I had to create my own versions of them.

Both of these tasks are beyond the scope of this article, but I'll share the general idea.

First, I added a folder called `build-helpers`. It includes a handful of Node scripts that perform specific operations. I run them before I build the Next.js site:

Quick little NPM protip: you can create pre-run scripts by using the `pre` prefix. When I run `npm run build` or `yarn build`, it will automatically run the `prebuild` script first, if defined. You can also run scripts afterwards with the `post` prefix.

Taking the RSS feed as an example, the process looks like this:

1.  Install the `rss` dependency: It'll handle the XML formatting for us.
    
2.  Collect all of the MDX files in the `pages` directory, using `fs.readdirSync`.
    
3.  Load the frontmatter, using gray-matter. Pluck out the relevant details (title, abstract).
    
4.  Filter out any unpublished posts (ones where `isPublished` is not set to `true`).
    
5.  Add each item to the feed, using the `rss` module
    
6.  Save the generated `.xml` file in `./public/rss.xml`
    

By storing it in `public`, I ensure that Next will copy it over to the `static` directory, and make it publicly-available. I also added this file to my `.gitignore`, since it's a generated file.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#design-and-assets)Design and assets

A bunch of folks have asked me how I created this little fella:

I wish I could take credit for it, but I didn't create it. I commissioned an artist to do it for me. We created several facial expressions, and two lighting modes (try toggling between light/dark mode using the toggle in the top-right!). Overall cost was around US$500, I believe.

Aside from that, I designed/built everything else you see on this blog. Design doesn't come naturally to me, but I've learned a few tricks over the years:

1.  When I've worked with designers, I've tried to learn from them. I asked questions like "how did you come up with this layout?" or "Why is this heading this color?". You can [build a design intuition](https://www.joshwcomeau.com/career/effective-collaboration/#develop-a-design-intuition) by trying to understand the rules and systems behind the designs you implement.
    
2.  For years, I never actually came up with anything from scratch. I'd search sites like [dribbble](https://dribbble.com/) and find 4-5 "references". I'd take the layout from one, the color scheme from another, the typography and spacing from a third. Learning to skillfully combine existing designs takes some practice, but it's a heck of a lot faster than learning to create compelling designs from scratch.
    
3.  There's a bit of a curse when it comes to design: if you spend 4 hours building something, you lose all objective perception. It's impossible to tell if it's good or not. For this reason, I always step away from a project for a day or two after I have a rough design in place. When I come back, I'll be able to tell if it's good or not.
    

My goal isn't to become a world-class designer — that would be the work of a lifetime, and an entirely separate career! But by putting in a bit of work and taking a few shortcuts, I've become competent enough at design to feel good about the things I build.

A lot of developers believe that you need to have some intrinsic artistic talent to become good at design, and I know that it's not true, because I'm a terrible artist 😅 If you're interested in becoming a better designer, be sure to [join my newsletter](https://www.joshwcomeau.com/subscribe/)—I'll be going much deeper into this stuff in the future.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#code-snippets)Code snippets

No developer blog is complete without syntax-highlighted code snippets. I have a couple different options for that on this blog.

Many Markdown processors allows us to create code samples with triple backticks (\`\`\`). We can specify the language for syntax highlighting as well (\`\`\`css).

With MDX, I map that syntax to a specific component, `StaticCodeSnippet`. It produces blocks like this:

Under the hood, this uses `prism-react-renderer` with a custom syntax theme.

Sometimes, though, I want the code to be "live-editable", and to showcase the result of that code. This is useful when I want to encourage the reader to experiment with the code, to learn how it works.

In those situations, I have a different component, `<Playground>`. It looks like this:

### Code Playground

To build this component, I forked [agneym's Playground](https://github.com/agneym/playground). It's a fantastic little utility. I did make some pretty substantial tweaks, mostly to the cosmetics and usability (the underlying rendering logic is mostly unchanged).

Here's what it looks like in the MDX:

This isn't the _best_ authoring experience: there's no syntax highlighting, and the indentation is funky. I often wind up writing the code in the playground itself, and then copying it over to the source.

One gotcha is that MDX doesn't like having blank lines in the middle of React elements. In the snippet above, I want a blank line between the two CSS rules. If I leave an actual blank line, MDX explodes with a hard-to-decipher error message. We can add an explicit newline with `\n`. Unfortunately, this creates _two_ blank lines, since the explicit `\n` is immediately followed by an actual linebreak. So I escape the linebreak with `\n\`.

Let's talk about some of the other less-than-ideal elements.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#downsides)Downsides

So here's a mistake I'm embarrassed to admit I've made more than once: sometimes, my "brand new" posts will have a "last updated" date from several months/years ago.

Here's why this happens: every blog post has a `publishedOn` date in the frontmatter, and optionally an `updatedOn` date as well. When I start writing a new post, I copy/paste some random old post to give me the frontmatter structure. If I don't explicitly remember to update the date when I publish the post (or whenever I update it), the wrong date is shown.

In an ideal world, `updatedOn` could be derived automatically, based on when the file was last modified. Operating systems can track when the file was last updated, so I ought to be able to nab that information from the OS when building the site.

Sadly, this doesn't work: my site isn't built on my local machine, it's built on Vercel's servers. They do a fresh clone of the files every time, so all of the files are brand-new according to the filesystem.

After publishing this post, Adam Collier shared how he [solved this problem](https://adamcollier.co.uk/blog/adding-an-updated-date-to-markdown-and-mdx-posts/), by editing the `.mdx` file at commit-time with lint-staged. I've implemented this in my own blog, and can confirm it works great. 😄

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#questions-and-answers)Questions and Answers

Before writing this post, I asked on Twitter if there was anything people were particularly curious about:

Unfortunately, there's no way for me to answer all of the questions I got 😅 Some of the questions were broad enough that I'd have to write an entire series of posts to be able to answer them!

But I can go through some of the most common questions.

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#how-do-you-organize-your-components)How do you organize your components?

Inside my `src/components` folder, there are about 150 components. These are the general "app-wide" components, stuff like `Logo` and `RainbowButton` and [`Boop`](https://www.joshwcomeau.com/react/boop/).

Each component gets its own directory:

I really like this pattern because it keeps the `src/components` directory relatively clear, while letting me create as many per-component files as I need. Creating this structure can be a pain, but I [use a command-line tool I created](https://www.npmjs.com/package/new-component) to make it super quick and painless.

I also have an `src/post-helpers` folder. Here I store all of the "one-off" components for specific posts, stuff like the `SpringMechanism` component we saw earlier. Ideally, I would colocate these components with their blog posts, but Next is pretty strict about what it allows in the `pages` directory (and `next-mdx-enhanced` requires you to keep your posts there).

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#whats-your-testing-strategy)What's your testing strategy?

I don't really have one 😅 Because this is a static site, there aren't many "critical flows".

I use Cypress on my course platform, though, and I'm quite happy with it!

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#how-do-you-come-up-with-article-ideas)How do you come up with article ideas?

As it happens, I wrote a fair amount about this in my newsletter recently! You can [read the issue in the archive](https://www.joshwcomeau.com/newsletter-issues/013/#my-authoring-process).

### [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#what-are-you-using-for-embedded-tweets)What are you using for embedded tweets?

Heh, so, I used to use the standard Twitter SDK. I found that it was slowing my whole site down, though.

I created a component, `FakeTweet`. That's what I'm using for this:

This is a very low-tech solution. I'm hardcoding all of the data. Here's what it looks like in the MDX:

The problem with this approach is that the data will become stale. People change their display names and user avatars, but my site won't keep up with the times. At some point, I'll write a build helper that will fetch data from the Twitter API using the tweet IDs.

## [

Link to this heading

](https://www.joshwcomeau.com/blog/how-i-built-my-blog/#going-deeper)Going deeper

For [reasons I've shared previously](https://www.joshwcomeau.com/blog/why-my-blog-is-closed-source/), this blog is closed-source, so unfortunately, there isn't a link to a Github repo for you to dig deeper. That said, I've enabled sourcemaps, so you can dig through the front-end code in your browser!

![Screenshot of the “sources” devtool, showing the code for one of the components](https://www.joshwcomeau.com/_next/image/?url=%2Fimages%2Fhow-i-built-my-blog%2Fsourcemaps.png&w=3840&q=75)

I'd encourage you not to get too focused on any particular aspect of this blog, though. Coming up with your own custom elements is one of the best parts of creating your own blog! Your blog is your own personal laboratory and playground: experiment with different ideas and see what you can come up with 😄 you'll have way more fun, and create something way more memorable and compelling.