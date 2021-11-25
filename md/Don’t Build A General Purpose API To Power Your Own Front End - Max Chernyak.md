> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [max.engineer](https://max.engineer/server-informed-ui)

> YAGNI

**TL;DR** YAGNI, unless you’re working in a big company with federated front-ends or GraphQL.

It’s popular in web dev nowadays to build a backend that serves JSON, and a frontend that renders the app. This is fine. I’m not the biggest fan, but it’s really okay. Except it’s not okay if you think that your backend needs to be designed like a generic public API. This will not save you time.

[#](#why-not)Why not?
---------------------

When you design a general purpose API, you have to figure out a bunch of annoying stuff.

1.  How to predict and enable all possible workflows
2.  How to avoid N+1 requests for awkward workflows
3.  How to test functionality, performance, and security of every possible request
4.  How to change the API without breaking the existing workflows
5.  How to prioritize API changes between internal and community requirements
6.  How to document everything so that all parties can get stuff done

And on the front-end side, there’s a bunch more:

1.  How to collect all the data needed to render a page
2.  How to optimize requests to multiple endpoints
3.  How to avoid using API data fields in unintended ways
4.  How to weigh the benefit of new features against the cost of new API requests

Do these really have to be your problems if you’re just making a backend for your frontend? Do you have to imagine every possible workflow, avoid N+1 request issues, test every request configuration, or deny yourself features when you know exactly what each page needs to look like? You can probably see where I’m going with this.

[#](#so-what-do-you-suggest)So what do you suggest?
---------------------------------------------------

I suggest you stop treating your frontend as some generic API client, and start treating it as a half of your app.

Imagine if you could just send it the whole “page” worth of JSON. Make an endpoint for `/page/a` and render the whole JSON for `/page/a` there. Do this for every page. Don’t force your front-end developers to send a bunch of individual requests to render a complex page. Stop annoying them with contrived limitations. Align yourselves. 🧘‍♂️

And in that JSON, actually render the page. Don’t render abstract models and collections. Render concrete boxes, sections, paragraphs, lists. Render the visual page structure.

```
{
  "section1": {
    "topBoxTitle": "Foo",
    "leftBoxTitle": "Bar",
    "linkToClose": "https://…"
  },
  "section2": {
    …
  }
}
```

This is similar but not quite the same as Server Driven UI[1](#footnote-12EY). Perhaps we could call it Server Informed UI.

[#](#how-is-that-better-exactly)How is that better exactly?
-----------------------------------------------------------

Have you seen that list of annoying decisions up there? For one, they are gone now. ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACQAAAAkCAYAAADhAJiYAAAACXBIWXMAAAsSAAALEgHS3X78AAABO0lEQVRYw82YzQ3DIAxGGYERGKEjZARGyAgZIRtkBEboBFVGYIIqI3DpPaWSkSyLUlLMT6V3qXp4NZ+NhRAFn8fzpTybx3pOwue7xSM9Ihdx5ceENSIRw3l0bSGTKYOZawltf8gENLeQKpAJxyc5hUyh0AlBZxNyDEKWU+hkZI8FvWV+UhVTI1SIhl31ylAyWz26LDk88aCzELQUtrLQjgNb+99nQY9EQiveRxGiTHCrGzgy11uoedfhUB8ovAYqg1mgYphqoZ4GCfWcCrVrLJM1GG8guMFRHqNdHbQDq1yuhkxq8yPI3KGWsfVDd8hMdIOkge7RafbXghbutZYVy94YdaN77fIKO5yQG01o7RnqGBJmVNO2z2FmXmPdt8FYguZ8fOAQCpVjeZ7hErpy/SQfsDiFQviXkie+N3nv15Pr2+u1AAAAAElFTkSuQmCC)

For two, you are now free to decide “I want page a” and then implement “page a” in the backend, and in the frontend. Super straightforward. ![](https://blotcdn.com/blog_ae6687516f4a44fb8370df95cf526289/_image_cache/c237b804-0a39-42cb-b7cc-a0629e3e52b5.png)

No more “what API workflows do we need to introduce to sort of make this page possible almost? 🤔”. You can keep “page a” dumb to only do what it needs to do. You test the crap out of “page a” for bugs, security, performance. You can even fetch everything for “page a” in a single big SQL query. You can cache the entire JSON payload of “page a”.

Frontend knows exactly what each field in “page a” payload is for. There are no discrepancies in field meanings. They represent exactly what frontend needs.

When a stakeholder tells you to change “page a” you will be able to literally go ahead and change “page a”, instead of spending meetings figuring out how your backend API could accommodate the change in “page a”. It’s not a choreographed conglomeration of API requests. It’s just “page a”. You have freed yourself from self-imposed limitations of your API.

Your business logic has now moved from being haphazardly split between frontend and backend into just backend. Your frontend can finally focus on presentation and UI. Your backend can finally focus on implementing exactly what’s needed. Kinda the goal, no?

[#](#have-you-actually-tried-this)Have you actually tried this?
---------------------------------------------------------------

Yes, I have tried this on a couple of production projects so far. One of them was personal, the other was a consistent multi-year refactoring effort in an existing company. The whole team was bought in, and it worked out well. The only problem we’ve encountered was that the front-end team has gotten increasingly bored. Nearly all business logic was taken away from them. At the same time, no “excitement” was added to the back-end team. It’s just gotten kinda boring all around. Somehow we all ended up talking more about the business than the code.

Feel free to stop reading here if you’re convinced. Next part is just responding to various rebuttals I keep hearing.

### [#](#but-i-want-my-front-end-team-to-have-freedom-or-i-want-my-front-end-to-be-decoupled)But I want my front-end team to have freedom! (Or, I want my front-end to be decoupled!)

Let’s be honest, your frontend doesn’t really have freedom. When they send you 7 requests to render a single page, that’s not freedom. It’s jumping hoops to meet basic requirements. As soon as requirements change, you probably going to need to change the backend anyway to accommodate it. The freedom is all accidental and mostly in the wrong places.

If you really want to give your front end team freedom, install them a GraphQL wrapper directly on top of Postgres and quit. ![](https://blotcdn.com/blog_ae6687516f4a44fb8370df95cf526289/_image_cache/fb91fc71-2c42-4a9a-96d7-2b7839fdf04f.png)

### [#](#but-we-actually-want-a-general-purpose-api-anyway-so-this-is-2-birds-with-1-stone-no)But we actually want a general purpose API anyway, so this is 2 birds with 1 stone, no?

No, you would not actually want to make this API public. You think you would, but when time comes, you’d be like “crap, maybe I shouldn’t”. These 2 APIs have very different reasons to change. Public API needs to enable the workflows of your clients. Private backend needs to enable the next whim of your product manager. Stop jamming sticks into your own bicycle wheels.

### [#](#but-how-will-i-reuse-the-logic-when-building-json-for-pages-i-reused-so-much-logic-in-my-crud-controllers)But how will I reuse the logic when building JSON for pages? I reused so much logic in my CRUD controllers!

If your programming language lets you reuse logic (it does), then you can reuse logic. Use mixins, composition, inheritance, whatever you got to work with. If you make yourself some good abstractions, then you will have an amazing time putting together pages from your LEGO blocks.

### [#](#but-we-can-reuse-this-api-for-the-mobile-app-too)But we can reuse this API for the mobile app too!

Your mobile app has a different set of pages with different info, structures, and reasons to change. You’ll save more time and sanity making another backend specifically for it. But hey, you can reuse a lot of your logic (see the previous paragraph).

### [#](#but-what-if-a-page-needs-a-partial-xhr-update-am-i-supposed-to-always-return-an-entire-page)But what if a page needs a partial XHR update? Am I supposed to always return an entire page?

No, it’s okay to make an endpoint that returns just something specific. You have my permission. Make endpoints for snippets of data for specific page sections or whatever. It’s okay. Render your React components from initial payload, then update them from XHR calls to these endpoints. But only introduce these endpoints when you need them on certain pages. These are exceptions, not the default.

### [#](#but-my-frontend-is-a-spa-so-it-almost-always-needs-data-snippets-not-entire-pages)But my frontend is a SPA, so it almost always needs data snippets, not entire pages

Those data snippets could still be provided as partial page structures, not generic resources. As long as your backend only serves the exact needs of your frontend, you’re good. ![](https://blotcdn.com/blog_ae6687516f4a44fb8370df95cf526289/_image_cache/ba0c8f92-0363-47ee-998e-873b458ed98a.png)

### [#](#but-im-building-a-site-builder-so-my-frontend-is-dogfooding-the-site-builder-api)But I’m building a site builder, so my frontend is dogfooding the site builder API

🗡 I dub thee a legitimate use case haver, congratulations!

### [#](#do-you-have-data-to-support-your-claims)Do you have data to support your claims?

I wish. It’s pretty hard to measure these kinds of things in our industry. Who’s gonna maintain 2 architectures for the same software for 3 years, and compare productivity between them? All I got is a mixed bag of personal experiences. Feels inductively justifiable. 🤷‍♂️