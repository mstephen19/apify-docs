---
title: Using DevTools
description: Learn how to use browser DevTools, CSS selectors, and JavaScript to collect data from a website.
menuWeight: 20.2
paths:
    - data-collection-basics/using-devtools
---

# [](#devtools-data-collection) Data collection with DevTools

We know the basics of HTML, CSS, JavaScript and DevTools and we can finally try doing something more practical - collecting data from a website. Let's try collecting the on-sale products from <a href="https://commerce-qd83plqbj-mstephen19.vercel.app/search/on-sale" target="_blank">this e-commerce website</a>. We will use CSS selectors, JavaScript, and DevTools to acheive this task.

## [](#getting-structured-data) Getting structured data from HTML

When you open up the <a href="https://commerce-qd83plqbj-mstephen19.vercel.app/search/on-sale" target="_blank">on-sale section of Morgan Webstore</a>, you'll see that there's a grid of products on the page with names and pictures of productsc. We will now learn how to collect all this information. Open DevTools and select the first product with the selector tool.

![Selecting an element with DevTools]({{@asset data_collection_basics/images/selecting-first-website-new.png}})

Now you know where to find the name of one of the products in the page's HTML, but we want to find all information about this product. To do that, in the **Elements** tab, hover over the elements above the one you just found, until you find the element that contains all the data about the selected product. Alternatively, you can press the up arrow a few times to get the same result.

![Selecting an element from the Elements tab]({{@asset data_collection_basics/images/selecting-container-element-new.png}})

This element is the parent element of all the nested (child) elements, and we can find it using JavaScript and collect the data. Notice that the element has an `aria-label` attribute, as well as multiple `class` attributes. This is where CSS selectors become handy, because we can use them to select an element with a specific class.

> Websites change and the structure or their HTML or the CSS selectors can become outdated. We'll try our best to keep this tutorial updated, but if you find that what you see on the website doesn't match this guide exactly, don't worry. Everything will work exactly the same. You will only have to use whatever you see on your screen and not in the screenshots here.

## [](#selecting-elements) Selecting elements with JavaScript

We know how to find an element manually using the DevTools, but for automated scraping, we need to tell the computer how to find it as well. We can do that using JavaScript and CSS selectors.

The function to do that is called `document.querySelector('some-selector')` and it will find the first element in the page's HTML matching the provided CSS selector. For example `document.querySelector('div')` will find the first `<div>` element. And `document.querySelector('p.my-class')` will find the first `<p>` element with the class `my-class`.

> You can find available CSS selectors and their syntax on the <a href="https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors" target="_blank">MDN CSS Selectors page</a>.

At the time of writing, the HTML element that contained all the data we wanted had an `<a>` tag and a `animated fadeIn` class, plus an `aria-label` attribute. This actually means that there were two classes applied to the element - `animated` and `fadeIn`. Neither of these seem reliable classes to go off of; however, each element also includes an `href` attribute which includes `/product/`, which we can use to our advantage.

```JavaScript
document.querySelector('a[aria-label][href*="/product/"]');
```

![Query a selector with JavaScript]({{@asset data_collection_basics/images/query-selector-new.png}})

> There are always multiple ways to select an element using CSS selectors. We always try to choose the one that seems the most reliable, precise, and unlikely to change with website updates. For example the `href` attribute can be assumed to always include `/products/`, as this is the path to view a specific product.
## [](#collecting-from-elements) Collecting data from elements

Now that we found the element, we can start poking into it to collect data. First, let's save the element to a variable so that we can work with it repeatedly and then print its text content to the console.

```JavaScript
const product = document.querySelector('a[aria-label][href*="/product/"]');
console.log(product.textContent);
```

![Print text content of an element]({{@asset data_collection_basics/images/print-text-content-new.png}})

As you can see, we were able to collect the data, but the format is still not very useful - there's a whole lot of weird data there that we probably don't need. For further processing (ex. in a spreadsheet), we would like to have each piece of data as a separate field (column). To do that, we will look at the HTML structure in more detail.

### [](#selecting-child-elements) Selecting child elements

In the [Getting structured data from HTML](#getting-structured-data-from-html) section, we were hovering over the elements in the **Elements** tab to find the element that contains all the data. We can use that to find the individual data points as well. After a bit more inspection, we discover that the product's title is a `<span>` tag with a parent `<h3>` element, and the price is held inside a `<div>` with a `class` attribute including the keyword of **price**.

> Don't forget that the selectors may have changed, but the general principle of finding them will always be the same. Use what you see on your screen.

![Finding child elements in Elements tab]({{@asset data_collection_basics/images/find-child-elements-new.png}})

The `document.querySelector()` function looks for a specific element in the whole HTML `document`, so if we called it with `h3`, it would find the first `<h3>` node in the `document`. Luckily we can also use this function to look for elements within an element.

There's a similar method called <a href="https://javascript.info/searching-elements-dom#querySelectorAll" target="_blank">`querySelectorAll()`</a>, which returns an array of all elements matching a specified selector - not just the first one. We will use this method to grab all the elements holding our sought after data.

> Learn more about `Arrays` <a href="https://javascript.info/array" target="_blank">in this tutorial</a>.

Let's find the parent element of all of the products, which matches the selector `div.grid.gap-6`, select it with `document.querySelector()`, then find all of the product elements.

```JavaScript
const grid = document.querySelector('div.grid.gap-6');

const products = grid.querySelectorAll('a[href*="/product/"]')

console.log(products);
```

There are 32 products on the page, so if we've done this correctly, a list of 32 product elements should be logged to the console.

![List child elements with JavaScript]({{@asset data_collection_basics/images/list-child-elements-new.png}})

### [](#collecting-data) Collecting data

The `products` array now contains all the elements we need, and we can access each one's data individually. Let's save the title and price of the first product into an <a href="https://javascript.info/object" target="_blank">object</a>. Those of you who know JavaScript will know that this is not the prettiest code ever written, but it is beginner-friendly and that's important here. We will also use the `.trim()` method to remove unnecessary whitespace from the results.

```JavaScript
const result = {
    title: products[0].querySelector('h3').textContent.trim(),
    price: products[0].querySelector('div[class*="price"]').textContent.trim(),
};

console.log(result);
```

![Print collected data to the Console]({{@asset data_collection_basics/images/print-website-data-new.png}})

If you were able to get the same result in your browser console, congratulations! You successfully collected your first data. If not, don't worry and review your code carefully. You'll crush the bug soon enough.

## [](#next) Next up

We have learned how to collect data about a few products from an e-commerce website. In the [next chapter]({{@link data_collection_basics/devtools_continued.md}}), we will learn how to collect all of it.
