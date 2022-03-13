---
title: API Scraping
description: Sniff out a site's API and use it to efficiently retrieve data.
menuWeight: 20.9
paths:
    - data-collection-basics/api-scraping
---

# [](#what) What is API scraping?

Simply put, API scraping is sniffing out a website's API endpoints, and retrieving the desired data directly from their API, as opposed to parsing the data from their hydrated pages.

> **Note:** In this tutorial, we'll be using <a href="https://soundcloud.com">SoundCloud's website</a> as an example target, but the techniques described here can be applied to any site.

# [](#advantages-disadvantages) Advantages of API Scraping

## Advantages

### 1. More reliable

Since the data is coming directly from the site's API, as opposed from the parsing of HTML content based on CSS selectors, it can be relied on more, as it is less likely to change. Typically, websites change their APIs much less frequently than they change the structure/selectors of their pages.

### 2. Configurable

Most APIs accept query parameters such as `maxPosts` or `fromCountry`. These parameters can be mapped to the configuration options of the scraper, which makes creating a scraper that supports various requirements and use-cases much easier. They can also be utilized to easily filter and/or limit data results.

### 3. Fast and efficient

Especially for <a href="https://blog.apify.com/what-is-a-dynamic-page/" target="_blank">dynamic sites</a>, in which a headless browser is required (which can sometimes be slow and cumbersome), scraping their API can prove to be much quicker and more efficient.

### 4. Easy on the target website

By using a website's API instead of sending a massive amount of traffic to their pages, the target website is less likely to suffer any potential performance issues.

## Disdvantages

### 1. Sometimes requires special tokens

Many APIs will require the session cookie, an API key, or some other special value to be included within the header of the request in order to receive any data back. For certain projects, this can be a challenge.

### 2. Potential overhead

For complex APIs that require certain headers and/or payloads in order to make a successful request, return encoded data, have rate limits, or that use modern technologies such as GraphQL, there can be a slight overhead in figuring out how to utilize them in a scraper.

# [](#detecting) Detecting whether or not there is data coming from an API

A lot of times, the response object of an API call can be found within a `<script>` tag in the DOM. Let's take a look at how to find these data objects.

Using our DevTools, we can inspect our <a href="https://soundcloud.com/tiesto/tracks">target page</a>, or right click the page and click "View Page Source" to see the DOM. Next, we'll find a value on the page that we can predict would be in a potential API response. For our page, we'll use the "Tracks" count of `845`. On the "View Page Source" page, we'll do <kbd>⌘</kbd> + <kbd>F</kbd> and type in this value, which will show all matches for it within the DOM. This method can expose `<script>` tag objects which hold the target data.

![Find the value within the DOM using CMD + F]({{@asset data_collection_basics/images/view-845.png}})

These data objects will usually be attached to the window object (often prefixed with two uderscores - `__`). When scrolling to the beginning of the script tag on our "View Page Source" page, we see that the name of our target object is `__sc_hydration`. Heading back to DevTools and typing this into the console, the object is displayed.

![View the target data in the window object using the console in DevTools]({{@asset data_collection_basics/images/view-object-in-window.png}})

This data object could be parsed directly from the DOM and used within a scraper,; however, its existence almost definitely confirms that there is an API endpoint that can be reached.

# [](#sniffing) Sniffing out API endpoints

In order to retrive a website's API endpoints, as well as other data about them, the "Network" tab within Chrome's (or another browser's) DevTools can be used. This tab allows you to see the all of the various network requests being made, and even allows you to filter them based on request type, response type, or by a keyword.

On our target page, we'll open up the Network tab, and filter by request type of "`Fetch/XHR`", as opposed to the default of "`All`". Next, we'll do some action on the page which causes the request for the target data to be sent, which will enable us to view the request in DevTools. The types of actions that need to be done can vary depending on the website, the type of page, and the type of data being returned. Sometimes, reloading the page is enough, while other times, a button must be clicked, or the page must be scrolled. For our example use case, reloading the page is sufficient.

_Here's what we can see see in the Network tab after reloading the page:_

![Network tab results after completing an action on the page which results in the API being called]({{@asset data_collection_basics/images/results-in-network-tab.png}})

Let's say that our target data is a full list of Tiësto's uploaded songs on SoundCloud. We can use the "`Filter`" option to search for the keyword "`tracks`", and see if any endpoints have been hit that include that word. Multiple results may still be in the list when using this feature, so it is important to carefully examine the paylods and responses of each request in order to ensure that the correct one is found.

> **Note:** The keyword/piece of data that is used in this filtered search should follow the same logic as the previous step - it should be a target keyword or a piece of target data that that can be assumed will most likely be apart of the endpoint.

After a little bit of digging through the different response values of each request in our filtered list within the Network tab, we can discover this endpoint, which returns a JSON list including 20 of Tiësto's latest tracks:

![Endpoint found in the Network tab]({{@asset data_collection_basics/images/endpoint-found.png}})

# [](#learning) Learning the API

The majority of APIs, especially for popular sites that serve up large amounts of data, are configurable through different parameters, query options, or payload values. A lot of times, an endpoint discovered through the Network tab will reveal at least a few of these options.

Here's what our target endpoint's URL looks like coming directly from the Network tab:

```
https://api-v2.soundcloud.com/users/141707/tracks?representation=&client_id=zdUqm51WRIAByd0lVLntcaWRKzuEIB4X&limit=20&offset=0&linked_partitioning=1&app_version=1646987254&app_locale=en
```

Since our request doesn't have any body/payload, we just need to analyze the URL. We can break this URL down into chunks that help us understand what each value does.

![Breaking down the request url into understandable chunks]({{@asset data_collection_basics/images/analyzing-the-url.png}})

Understanding an API's various configurations helps with creating a game-plan on how to best scrape it, as many of the parameters can be utilized for easy pagination, or easy data-filtering. Additionally, these values can be mapped to a scraper's configuration options, which overall makes the scraper more versatile.

Based on our observations, we can modify the URL and utilize the "`limit`" option to return 200 songs in one request, instead of just 20:

```
https://api-v2.soundcloud.com/users/141707/tracks?client_id=zdUqm51WRIAByd0lVLntcaWRKzuEIB4X&limit=200
```

# [](#challenges) Dealing with headers, cookies, and tokens

Unfortunately, most APIs will require a valid cookie to be included in the "`cookie`" field within a request's headers in order to be authorized. Other APIs may require special tokens, or other data that validates the request.

Luckily, there are ways to retrive and set cookies for requests prior to sending them, which will be covered more in-depth future Scraping Academy modules. The most important things to know at the moment are:

1. For sites that heavily rely on cookies for user-verification and request authorization, certain generic requests (such as to the website's main page, or to the target page) will return back a (or multiple) "`set-cookie`" header(s).
2. The `set-cookie` response header(s) can be parsed and used as the `cookie` header in the headers of a request. A great package for parsing these values from a response's headers is <a href="https://www.npmjs.com/package/set-cookie-parser">`set-cookie-parser`</a>

Other APIs may not require a valid cookie header, but instead will require certain headers to be attached to the request which are typically attached when a user makes a "real" request from a browser. The most commonly required headers are:

-   `User-Agent`
-   `Referer`
-   `Origin`
-   `Host`

Headers required by the target API can be configured manually in a manner such as this, and attached to every single request the scraper sends:

```JavaScript
const HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 YaBrowser/22.1.0.2500 Yowser/2.5 Safari/537.36',
    Referer: 'https://soundcloud.com',
    ...
}
```

However, a much better option is to use either a custom implementation of generating random headers for each request, or to use a package such as <a href="https://www.npmjs.com/package/header-generator">`header-generator`</a> to automatically do this.

For our SoundCloud example, testing the endpoint from the previous section in a tool like <a href="https://www.postman.com/">Postman</a> works perfectly, and returns the data we want; however, when the "`client_id`" parameter is removed, we receive a `401 Unauthorized` error. Luckily, the Client ID is the same for every user, which means that it is not tied to a session or an IP address (this is based on our own observations and tests). The big downfall is that the token being used by SoundCloud changes every few weeks, so it shouldn't be hardcoded. **A very strange case indeed**.

# [](#extra-challenges) Extra challenges

> **Note:** these concepts will be covered more in-depth later on, but it is good to be aware of them now.

## 1. Different data formats

APIs come in all different shapes and sizes. That means every API will vary in not only the quality of the data that it returns, but also that format that it is in. The two most common formats are `JSON` and `HTML`.

JSON responses are the most ideal, as they are easily manipulable in JavaScript code. In general, no serious parsing is necessary, and the data can be easily filtered and formatted to fit a scraper's output schema.

APIs which ouput HTML are generally returning the raw HTML of a small component of the page which is already hydrated with data. In these cases, it is still worth using the API, as it is still more efficient than making a request to the entire page; even though the data does still need to be parsed from the HTML response.

## 2. Encoded data

Sometimes, a response will look something like this:

```JSON
{
    "title": "Scraping Academy Message",
    "message": "SGVsbG8hIFlvdSBoYXZlIHN1Y2Nlc3NmdWxseSBkZWNvZGVkIHRoaXMgYmFzZTY0IGVuY29kZWQgbWVzc2FnZSEgV2UgaG9wZSB5b3UncmUgbGVhcm5pbmcgYSBsb3QgZnJvbSB0aGUgQXBpZnkgU2NyYXBpbmcgQWNhZGVteSE="
}
```

Or some other encoding format. This example's `message` has some data encoded in <a href="https://en.wikipedia.org/wiki/Base64">Base64</a>, which is one of the most common encoding types. For testing out Base64 encoding and decoding, you can use <a href="https://www.base64encode.org/">base64encode.org</a> and <a href="https://www.base64decode.org/">base64decode.org</a>. Within a project where base64 decoding/encoding is necessary, a package such as <a href="https://www.npmjs.com/package/base-64">`base-64`</a> can be used to encode and decode Base64 data within Node.js code.

## 3. Different types of APIs

The vast majority of APIs our there are standard REST APIs that have different endpoints. Even though this is the case, it is important to be aware that newer API technologies such as <a href="https://graphql.org/">GraphQL</a> are becoming more popular, and therefore more utilized in modern web applications.

A GraphQL API works differently from a standard API. All requests are `POST` requests to a single endpoint - typically `https://targetdomain.com/graphql`. Queries are passed as a payload, and are very specific (which can be difficult to deal with). Additionally, GraphQL is a language in of itself; therefore, one must at least skim their documentation for a light understanding of the technology in order to be able to effectively scrape an API that uses it.