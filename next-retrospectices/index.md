---
title: "A Next.js Application Retrospective"
---
I've spent the last couple of years developing an SSR application with Next.js and Apollo. The project features a mixture of job boards, a large amount of content (from a CMS) and user profiles. I thought I'd share some of the challenges, mistakes and thing's I wish I knew before I started.

Some of these to you might be quite obvious, however what I listed here was really important to the success of our project. Hopefully it will be of use to some.

# tldr;

- [Do you really need to SSR?](#do-you-really-need-to-ssr)
- [Monitor Performance Early](#monitor-performance-early)
- [getInitialProps Optimisation is Vital](#-raw-getinitialprops-endraw-optimisation-is-vital)
- [Apollo Cache Key Optimisation is Vital](#apollo-cache-key-optimisation-is-vital)
- [Watch Out for Styled Component Nested Props](#apollo-cache-key-optimisation-is-vital)
- [async/await in getInitialProps](#-raw-asyncawait-endraw-in-raw-getinitialprops-endraw-)
- [Consider Building a Version Notification System Early On](#consider-building-a-version-notification-system-early-on)

# Do you really need to SSR?

The first hurdle was determining if we really needed server side rendering. Obviously there are many pros and cons to analyse. It's really important to weigh these options and understand the trade-offs between a static, dynamic or server side build.

For us, SEO was a determining factor. We needed to display source HTML for search engines, as well as managing meta and redirects. Alternative methods, such as pre-generated content just wasn't viable, as we had huge amounts of multi-tenant content that's published and changed frequently.

You should ask yourself if the benefits of SSR (eg: SEO, request/response control) outweigh the cons (eg: time to first byte, complexity, server infrastructure).

As a general guideline, I try to favour static generated sites for performance reasons. Study [`getStaticPaths`](https://nextjs.org/docs/basic-features/data-fetching#getstaticpaths-static-generation) to see if it can work for you.

# Monitor Performance Early

This aspect is not tied to SSR rendering, but an important fundamental. It's too easy to make a decision and be completely unaware that you've made a huge impact on performance.

For example, you've got a requirement that needs to support timezones. You use moment.js, build your feature, test it, deploy it. Boom. Easy. However, you didn't realise that you accidentally used all locales, which means 4MB of uncompressed JSON is now being distributed to your users.

This mistake may be obvious to you, but maybe not to a team member.

So many mistakes could of been prevented by implementing a [fitness function](https://www.thoughtworks.com/radar/techniques/architectural-fitness-function) that analyses performance as part of the CI pipeline. Automatic budgets on time to first byte, bundle size and HTML page size are __really important to prevent performance blunders being deployed to production__. Use something like [Google Lighthouse](https://github.com/GoogleChrome/lighthouse-ci) to track those metrics.

This optimisation will help find many problems mentioned later in this article.

# `getInitialProps` Optimisation is Vital

Everything that you pass from `getInitialProps` gets hydrated. It's __printed__ directly to HTML, which means the user has to retrieve and parse it first __before anything else can happen__.

Always keep this is the back of your head. __Do you need that data?__ It's very easy to pass a large amount of data that isn't used, forcing users to download and wait for a whole bunch of data they don't need.

This is something that should be tracked in the CI pipeline.

# Apollo Cache Key Optimisation is Vital

When using Apollo, you should have a basic awareness on [how normalisation works](https://www.apollographql.com/docs/react/caching/cache-configuration/#data-normalization). Apollo will attempt to break up and normalise objects on a type and ID. Failing that, it will attempt to store the object based on how it's queried.

Apollo will try to pass the cache during the hydration process, by __printing it to HTML__. This is a huge optimisation opportunity for SSR. Without effectively normalised data, the HTML size can really grow quickly.

A lot of our content uses an `entityId` unique identifier, instead of the default `_id` or `id` identifier. The cache passed via HTML was huge in some cases. By [implementing a cache key](https://www.apollographql.com/docs/react/caching/cache-configuration/#customizing-identifier-generation-by-type), large amounts of raw HTML was trimmed.

This is something that should be tracked in the CI pipeline.

# Watch Out for Styled Component Nested Props

When using styled components, you can pass dynamic props to a component:

```jsx
import styled from "styled-components"

const Style = styled.div`
  border: 1px solid blue;

  .title {
    color: ${props => props.color};
  }
`

const Card = props => (
  <Style>
    <h1 className="title" color="blue">Card</h1>
  </Style>
)
```

This feature is powerful, however because the prop is on a nested selector, this caused __large amounts of duplicated styles to be printed directly to HTML__, which again, forced users to download and wait for data they didn't need.

__Be very vary of using props on nested styles__.

This can be resolved by using an additional styled component:

```js
<Title className="title" color="blue">Card</Title>
```

Or by using plain old CSS (my preferred approach):

```css
.title {
  color: attr(data-color);
}

<h1 className="title" data-color="blue">Card</h1>
```

# `async/await` in `getInitialProps`

Using `async`/`await` in `getInitialProps` will block the entire request until the response has been resolved.

```js
Content.getInitialProps = async () => {
  const response = await fetch() // ...
  const meta = response.data.meta
  return { meta }
}
```

Avoid doing this as much as you can, be __aware of the cost__. Every millisecond waiting for that response will an extra millisecond on the time to first byte.

# Consider Building a Version Notification System Early On

So any headaches were caused because a lot of user's __leave tabs open__ and __do not refresh the page__. The Next.js application is cached in memory over client side navigation, and may not receive a new version until the page has been refreshed. If your application communicates with an API, this situation gets complicated.

## Example

A new version of the CMS removes a data type from the content API, which the frontend relies on. The change gets deployed on both platforms. However, the user has the original version in the browser. They navigate to a page that relies on that now removed data type, theres an error.

Yes, there are many ways to handle this error and could be alleviated with API versioning. However, the problem is complex, you may easily find yourself fall into this trap accidentally.

It may not be immediately obvious to the user that they need to refresh, __especially if the error is silent__. The situation becomes worse if that request is critical, such as authentication.

Notifying users that a new version is available can really help improve the situation. This solution may not be for you, but a notification system is a simplistic implementation that can help drastically.

# That's All

If you got this far, thanks for taking the time to read my first published article. Hopefully this helps some avoid the traps and pitfalls made.


