# Node Web Scraper Cookbook

The web is a wealth of information, not all of it is easily accesible in a "data format" like RSS or an API. Web scrapers can turn inaccessible information into actionable inputs.

In this repo I'll try to provide some examples of various strategies for web scraping. I'll also document the various tools and packages I find useful along with various other tips. This repo is primarily concerned with web scraping using [Node.js][node].

**First tip**: Always `curl` the target site to understand what comes back, the `-O` and `-L` flags are your friends.

## Overview

Web scraping according to [Wikipedia][wiki] is:

> a computer software technique of extracting information from websites.

I've started writing scrapers anytime I run into a site that meets any of the following criteria:

- The site does not provide a RSS for content that _should_ have a RSS feed (looking at you [Youtube][yt]!)
- The site is selling tickets or having an auction and I'd like to know immediately whenever the status changes.
- The site is so damn awful to look at and covered with so many ads (seriously, I made billions working from home, find out how!!!) that the content is impossible to get to.

## The Happy path

If you followed the first tip and `curl`ed the target site and saw the information you were after, then you're almost done! All that's left is to use [request][request] + [cheerio][cheerio] to automate the retrieval and parsing of the information for you. [Request][request] is _the_ HTTP client for Node. It has a _ton_ of options, but we will only need the basics to scrape. [Cheerio][cheerio] is the best Node module I've ever come across. It's _awesome_. Cheerio provides a "[jQuery][jq]-like" experience with any DOM string. Just load it up and you can start doing stuff like this:

```javascript
//html will probably come from a `request` response
//it could also come from a file, `fs.readFileSync('dump.html').toString();`
const $ = cheerio.load(html);

$('.products').each((i, e) => {
    const text = $(e).text();

    if (text === 'Xbox One') {
        this.sendAlert();
    }
});
```

Pretty great eh? Looks like just like typical jQuery stuff. Parse and filter to your heart's content. Did I mention cheerio is insanely fast too? It is.

## JS-only content

Scraping is all easy and painless with sites that work without JS (like the scenario above :point_up:). This is not the case for sites behind Javascript authentication or sites that are Javascript applications. Thankfully we have [headless Chrome][headless]. Headless Chrome let's you run an instance of Chrome (headlessly). One approach I've used with success is to limit the amount of work I do inside headless. I'll navigate to the page where my info is and then dump the entire page's worth of HTML to a file and pass it all off to [cheerio][cheerio] which handles traversing and parsing that markup _much_ better than headless Chrome. I've also used it to pull the relevant authentication information I need to then make API requests via normal `axios` or `curl` commands.

## ❤ Wordpress

Wordpress runs half the web (or that's how it seems). This is great news for scrapers since most WP installs [have several RSS feeds][wp] to pull data from. So if you come across a site and you figure out it's a Wordpress install, check if any of the [various built-in feeds][wp] work.

For parsing a RSS feed, [saxjs][sax] is the best module to use.

## Actions?

So you've scraped the site and you have the information, now what? Well now you can send yourself a SMS ([Twilio][sms] is awesome and very cheap) or an email ([Sendgrid][mail] has a great free plan) with the relevant info. Or save the info in a database (e.g. [SQLite][sql]). It's up to you at this point.

## Deployment

Now we need to setup how often our scraper will run and where we will host it.

Scrapers are typically very light on resource usage, I've paired several on the same VPS as my [blog][blog] without a single issue. If you don't have a server to host a scraper, I'd recommend spending a few bucks on a ["Lowend" box][box]. Even `128MB` of RAM is plenty.

I've found using [Ansible][ansible] to be nice for deploying and setting up scrapers. It makes pushing updates (`HTML` changes pretty often) easy and painless.

I use [cron][cron] to run all my scrapers at set intervals.

---

**Important** please be courteous of _all_ websites and throttle your scraper to only hit a site a few times per hour. Not only is it the respectful thing to do, but it'll also prevent you from getting your IP banned. The most often I have a scraper running at the moment is once a hour.

---

Speaking of cron, Ansible has a great [cron module][acron].

## Software Tools

Since this entire cookbook is based around using Node for scraping the following packages are all Node related. Two tools I haven't mentioned yet that deserve a mention are [loadsh][_] and [asyncJS][async]. Lodash provides some great utility methods to work with page data. [AsyncJS][async] provides a way to speed up and organize your code if you find yourself sending multiple `request`s at once or doing other asynchronous programming.

- [request][request] - HTTP client
- [cheerio][cheerio] - DOM parser
- [headless Chrome][headless] - "headless" Chrome
- [lodash][_] - utility library
- [async][async] - tools for dealing with asychronous code
- [saxjs][sax] - XML (rss/feed) parser
- [cron][cron] - schedule jobs
- [sqlite][sql] - database of choice, very fast and low resource usage
- [noodle][noodle] - a Node.js server and module for querying and scraping data from web documents.

## Services

- [Twilio][sms] - SMS and VOIP
- [Sengrid][mail] - API for sending email

## Helpful sites

- [W3 RSS Validator][w3] - RSS validator
- [RSS Validator][rss] - RSS validator
- [User agent strings for bots][bots] - Wikipedia guidelines for bots

## Examples and going forward

I plan to keep this cookbook up to date with new tools and methods as I find them. I also plan to include some additional "sample projects" as I find time to commit them. For now, you can reference my [ESPN Scraper + RSS feed][espn] repo as an example of a `request` + `cheerio` + `sqlite` project.

[headless]: (https://github.com/GoogleChrome/puppeteer)
[request]: https://github.com/request/request
[cheerio]: https://github.com/cheeriojs/cheerio
[wiki]: https://en.wikipedia.org/wiki/Web_scraping
[sax]: https://github.com/isaacs/sax-js
[sms]: https://www.twilio.com
[_]: https://lodash.com
[async]: https://github.com/caolan/async
[w3]: https://validator.w3.org/feed/
[rss]: http://feedvalidator.org
[bots]: https://meta.wikimedia.org/wiki/User-Agent_policy
[mail]: https://sendgrid.com/pricing
[yt]: https://ytrss.co
[cron]: http://alvinalexander.com/linux/unix-linux-crontab-every-minute-hour-day-syntax
[ansible]: http://docs.ansible.com/ansible/intro_getting_started.html
[wp]: http://codex.wordpress.org/WordPress_Feeds
[casper]: http://casperjs.org
[capy]: https://github.com/jnicklas/capybara
[jq]: http://jquery.com
[sql]: https://github.com/mapbox/node-sqlite3
[blog]: https://adamsimpson.net
[box]: http://lowendbox.com
[acron]: http://docs.ansible.com/ansible/cron_module.html
[node]: http://nodejs.org
[espn]: https://github.com/asimpson/espn-scraper-to-rss
[noodle]: http://noodlejs.com
