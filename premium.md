The **Premium Podcast Protocol** is a simple, open standard for distributing premium podcasts by slightly extending a standard podcast RSS feed.

This protocol gives podcasters the option of charging for a podcast if they wish, while still creating an open ecosystem of podcast apps.

Without an open standard for authenticated podcasts, proprietary podcast apps might take over if it turns out that a premium model is something listeners and podcasters like. Lots of people prefer an ad-free listening experience but also care about an open world of podcasting.

There's multiple models this simple protocol can support:

- A podcast that is entirely paid, with an optional sample episode or audio trailer.
- A podcast that contains occasional premium episodes.
- A podcast that allows you to upgrade to a paid ad-free version of the same episodes.
- A podcast that provides paid access to back episodes, but the most recent episodes are free.
- A podcast that has exclusive episodes gated by something other than money.
  - For example: a podcast associated with a video game where the top scorers get exclusive content.

Also, don't forget about video podcasts. Maybe premium podcasts with video could be a nice open replacement for YouTube.

> **Sidebar: Don't premium podcasts already exist?**
>
> Sort of. Most podcast players support HTTP Basic Auth for podcasts, but those podcasts can't be discovered by searching within apps, there's not a unified login flow, and it's hard to distribute previews or sample episodes. For this reason, they're not as widely used as they could be.

## The Idea

This protocol works by splitting your podcasts into two pieces: the _public feed_ and the _private feed_.

Anyone can listen to the episodes in the public feed, and they can optionally also authenticate to access the private feed too. Podcast players are free to interleave these two streams of episodes in whatever ways make sense.

#### The Public Feed

The _public feed_ is the public face of your podcast. It's a normal podcast with two important differences:

1. You only list episodes you want everyone to be able to listen to. For example, there might be a sample episode or a special introductory episode pitching the podcast.

2. This feed contains special markup to allow podcast apps to find and authenticate to the _private feed_.

#### The Private Feed

The _private feed_  contains all your premium episodes and is protected by HTTP Basic Authentication.

The server your podcast is hosted on manages credentials however it wants, and is responsible for ensuring that the private feed URL responds with an HTTP 403 or 404 error when credentials are invalid.

## The Experience

1. You subscribe to a podcast in your favorite podcast player.
2. You listen to the free episodes (or trailers or sample episodes).
3. You decide you'd like to access the premium content, so you click a "Get Premium Episodes" button in your player.
4. You're sent to the login or signup flow for whatever service is handling the premium payments.
5. You're returned to your podcast player with the premium episodes unlocked.

A podcast player might choose to display premium content mixed in with the public content, in a seperate tab/screen, or something more creative.

## The Markup
Here's an example premium podcast RSS feed:

```xml
<channel>
  <title>My Great Podcast</title>
  <link>https://cast.example/publicfeed</link>
  <itunes:subtitle>Just the most fun audio show!</itunes:subtitle>
  <itunes:author>Jane Doe</itunes:author>

  <auth:private-link>https://cast.example/privatefeed</auth:private-link>

  <item><!-- Sample Episode --></item>
</channel>
```

As you can see, there's a new `auth:private-link` tag. This is the URL of the private feed.

Since the private feed uses HTTP Basic authentication, the podcast app can only load it if it has saved credentials.

If there are no saved credentials, the app will only have access to episodes in the public feed. It will probably also show a big juicy "Listen to Premium Episodes" button. Pressing that button attempts to load the `auth:private-link` and prompts for HTTP Basic Credentials. When the user gets those correct, the podcast app remembers them and uses that to load up the premium feed.

### Fancier Login and Signup Flows

Not all services are appropriate for direct login with HTTP Basic Auth. For example, you might want to login using OAuth or have some kind of two-factor authentication for your super secret podcast.

For that, we can optionally include a second new tag:

```rss
<auth:login-link>https://cast.example/login</auth:login-link>
```

This enables the "Get Premium Episodes" button of the podcast app to launch a more complex login flow.

The flow starts by the app redirecting to or opening an interface for the `auth:login-link`. When it does so, it will append a URL parameter `redirect_uri` to the login link.

> In our example, that would look like `https://cast.example/login?redirect_uri=https%3A%2F%2Fapp.example%2Fdone`.

That `redirect_uri` could be anything, it's up to the app initiating the login flow.

Whatever pages or OAuth flows are neccessary can be completed by the server hosting the login link. When the server is done, it redirects to the `redirect_uri` that it received, adding two new URL parameters: `username` and `password`.

> In our example, that would look like `https://app.example/done?username=foo&password=bar`.

The app then stores the credentials and uses it to authenticate to the `auth:private-link`. If that link stops working, it can ask the user to login again.

## Potential Issues

- Does opening up these kinds of options create bad incentives for content creators or networks of content owners?
- Doesn't really create a unified billing infrastructure, so people are still probably dealing with tons of accounts.
- It could be sketchy to pass the basic auth username and password back through the redirect url in the advanced login case. Though, over HTTPS I think it should be fine for this use case? Should be checked. It should be as simple as possible for it to catch on with podcasting apps.

## Potential Extensions

- Allow a way of specifying that a premium episode should "replace" a non-premium episode (such as if the premium version of the show is ad-free).
