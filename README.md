# Universal Listen ID (ULID)

<small>Version 1.2 by [Dan Benjamin](https://danbenjamin.com), [Dave Jones](https://github.com/daveajones)</small><br>
<small>February 1st, 2025</small>

<br>

## Introducing ULIDs
A Universal Listen ID (ULID) is a simple way to get accurate and trustworthy download, stream, and subscriber counts for podcasts without compromising privacy or paying exorbitant fees to financially motivated corporations. ULIDs are private and fully transparent by design, create zero overhead, and are easy to implement for both podcast hosts and apps.  A nice overview of the problem that the ULID is attempting to solve can be found in [this](https://blog.podcastindex.org/html/TimeToHangUptheEgosa-kFtkXVfBmdDVR1w34w3Q1x01WKSn9W.html) blog post.

If your hosting service or podcast app supports ULIDs, please let us know. We are currently building lists of supporting apps and services for publication [here](supporting_platforms.md).  Just submit a pull request!

<br>

## What Are ULIDs?
Structurally, a ULID is simply a [UUIDv4](https://www.uuidtools.com/generate/v4) value. This is nerd-speak for a 128-bit structure that is highly unlikely to ever repeat or create a collision, which means they are unique, one-of-a-kind strings of numbers and letters generated by computers.

They look like this:  `43b811dc-3697-4361-85ef-489bf9bf2deb`

This simple, random value gives the podcast industry an open, trustworthy and free standard for tracking and measuring downloads without compromising privacy. In fact, ULIDs extend privacy because they make relying on non-private data — such as IP addresses — unnecessary.

The best thing about ULIDs is that they are fully transparent to the podcast listener. They do not change or affect the way listeners consume and enjoy podcasts in any way. Instead, they add a layer of privacy and uniqueness to each download, ensuring an accuracy of measurement at least as good as what is available with IP address tracking.  And, it futureproofs for a time when IP addresses become more obscured.

<br>

## How ULIDs Work
When a podcast app is ready to download or stream a new episode, it generates a new ULID and sends it along as part of the request to download that episode, tagging the request in a globally unique way. The app then hangs on to that ULID for a maximum of 10 days and sends it along with each subsequent request for that same episode from the same user.  When the listener deletes the episode or finishes the stream (or if 10 days has passed) the app throws away that ULID forever, never to be used again.  The ULID is a random string that represents a specific user retreiving a specific episode during a specific 10 day time slice.

<br>

## I Have an App or Service, How Do I Implement ULIDs?
Our suggested implementation is straight forward, and is intended to be used the same way across all apps and services which wish to support the ULID specification. It involves the use of the `_ulid` URL parameter, as described below.

<br>

### App Developers
When your app requests a stream or download of a podcast episode, it should perform the following steps:

1. Generate a new case-insensitive 128-bit string in the standard UUIDv4 format, such as: `43b811dc-3697-4361-85ef-489bf9bf2deb`
2. Append that string as a URL parameter named `_ulid` to the download request, or streaming range request the app makes for a specific episode’s audio file. Here's an example:
   `GET https://example.com/path/to/audio.mp3?_ulid=43b811dc-3697-4361-85ef-489bf9bf2deb`
3. When the download is deleted, the stream is completed or 10 days has elapsed, throw the ULID away. If addional downloads are initiated, a new ULID value is created, attached, and then discarded in the same fashion.

### Hosting Service Developers
When your service processes new downloads from a user agent known to be ULID compliant:

1. Look for unique `_ulid` parameters when processing download records.
2. When one is found, record just one download/stream, throwing away any additional requests with the same `_ulid` parameter.
3. If one isn’t found, do the normal processing you’ve been doing all along.

That’s it. That’s all there is to it. But please be sure to let us know so we can update our supporting apps and hosting services [list](supporting_platforms.md).

<br>

## Current Data Is Unreliable
Since the early days of podcasting, podcasters have relied on a combination of user agents and IP addresses as the primary method for measuring listener data (like streams, downloads, and subscribers). Unfortunately, this method is flawed.

At the heart of the issue is the range-request, the technology used by servers in order to speed up the delivery of content across the Internet. When a podcast app requests a file from the server, the server can respond with multiple, simultaneous streams of data rather than just one. This decreases the time it takes for a download by the app but also creates a tricky problem: each request will appear in the server’s log as a download, when in fact it isn’t.

So, while range-requests are incredibly beneficial, they have the unintended side-effect of creating potentially dozens of download records with each file request, greatly exaggerating download numbers. Current methods attempt to mitigate this through a set of rules, created primarily by advertisers, to try and determine which requests are range-requests and which are not. Further, they try to determine which are unique through an equally flawed system of tests, checks, and assumptions.

<br>

## Listener Privacy Is Being Violated
IP addresses reveal a lot of information about a listener. With just your IP address, your geographic location, often down to the city block, can be easily identified. IPs can also reveal your personal Internet service provider, your employer, and much more. When combined with your podcast app’s user agent and the time of day, a whole lot can be learned about not just your listening habits, but your behaviors, your schedule, and much more.

> IP addresses have, over time, become a treasure trove of PII (personally identifiable information). It’s truly startling how much you can know about
> a person simply from their IP address. The fact that podcast stats rely so heavily on the same data point which is now in the crosshairs of powerful,
> privacy minded platform builders, should wake up the entire industry.
> <small><br>--[Dave Jones, Podcastindex.org](https://blog.podcastindex.org/html/TimeToHangUptheEgosa-kFtkXVfBmdDVR1w34w3Q1x01WKSn9W.html)</small>
  
Every time you download a podcast episode, your IP, and therefore your private information, is being stored, examined, analyzed, and recorded. A profile about you is being built, slowly, every time you listen.

<br>

## Current Methods Are Flawed
According to currently accepted measurement standards, enough of an episode must have been downloaded by the app to play for one minute in order for it to be considered valid. This makes sense, because we want to be certain that an episode has actually been retrieved by an app, not just checked for changes by a server or crawler, for example.

However, the episodes’s encoding bitrate, ID3 tags, embedded images, chapters, and more all directly affects filesize and vary greatly from episode to episode and podcast to podcast. These calculations must be performed on every single episode across every podcast, just to make this simple determination. And even when this is done correctly, it still doesn’t guarantee that the file was really downloaded by an actual person. We still have to examine IP addresses, check download timecodes, and more.

And then there’s the issue of networks. Say everyone in your team, or on your floor, in your coffee shop, or in your company, all subscribe to the same podcast. When a new episode drops, all of your podcast apps and clients will attempt to download it at the very same time, often at the same exact moment if the same app is used, which is typical. Because all of you are on the same network, you’ll be using the same Internet gateway, which means you’ll all be identified as coming from the same, identical IP address, thus nuking the unique IP address requirement, and making what could be several, dozens, or even thousands of downloads appear to be just one. That’s a big problem.

<br>

## A Note About Fraud
Could someone game the numbers by generating fake downloads with unique ULIDs? Unfortunately yes. But this can already be done using the existing measurement systems. Fraud detection is always part of the game in the world of digital advertising and attribution. The aim isn’t to design a system that is un-gameable, because that’s just not possible. Instead, the goal should be to create a system that’s simple and transparent enough to make fraud detection a straightforward process.
