---
layout: post
title:  "5 Tips to Distinguish Good and Bad Locators"
date:   2019-01-07 18:14:37 +0700
categories: blog
excerpt_separator: <!--more-->
---
This article aims to share the best tips in capturing web elements crystalized from our experience working with multiple web applications and multiple clients. Hopefully after a pleasant read...
<!--more-->
## Why reading these tips?

Warmest greetings from the **POM Builder** team! We hope you’re doing well. 

To make life easier for hundreds of thousands of automation engineers, our team has come up with an idea: why not start at the beginning? As you might already know,  Test Automation usually begins with mapping UI elements on the web app under test. The result of this mapping process is called element locators or selectors. Some refer to the process as capturing web elements. Only after we have those element locators in place, we start interacting with the web elements in our test cases. 

If we fail at capturing good locators for those web elements, very soon we’ll regret not doing a better job from the beginning. This seemingly easy task tells experienced testers apart from inexperienced ones. Those experienced testers seem to have a knack for quickly judging whether a locator is reliable or not.

That said, we believe that such technique can certainly be taught and learned. This article aims to share the best tips in capturing web elements crystalized from our experience working with multiple web applications and multiple clients. Hopefully after a pleasant read, you’ll become better in picking the most robust locators for your project. 

## Tip #1. Long texts are usually BAD
Imagine that you are writing functional tests for the newsletter subscription feature of a web page, something like [this page](https://www.time-to-change.org.uk/email-signup). People can select which types of emails to subscribe to by ticking various checkboxes. 

Your automated test will fill all the required fields, tick one option in the below picture by clicking on the corresponding `<label>` tag, click “Submit” and check whether a confirmation email would be sent to the registered email address. 

<a href="https://imgur.com/0GHF0Dl"><img src="https://i.imgur.com/0GHF0Dl.png" title="source: imgur.com" /></a>

A lengthy text describes each option. Let’s say we want to capture the checkbox described as **"General updates - for regular emails about all areas of our programme"**. A novice tester would immediately pick the most self-explanatory locator as below.

    XPath = //label[contains(.,'General updates - for regular emails about all areas of our programme')]

Although self-explanatory, this locator is not ideal since the lengthy text is probably fragile in the long run. It might change any time without notice. Let’s say for the sake of American English, the developer decides that it’s better for the whole world to spell “programme” as “program”. Your automated tests will definitely fail in groves.

A closer look at the DOM structure would reveal that we can approach this problem in a much better way. Lucky for us, the `<div>` tag which is also the parent of the label has a reliable @class attribute. Its current value is **form-item form-type-checkbox form-item-submitted-civicrm-1-contact-1-other-group-948**. The other `<div>` tags differentiate from this one only by the last numbers, e.g. **form-item form-type-checkbox form-item-submitted-civicrm-1-contact-1-other-group-951**. 

<a href="https://imgur.com/lHE5KLS"><img src="https://i.imgur.com/lHE5KLS.png" title="source: imgur.com" /></a>

Thus we can safely take advantage of this @class attribute and parts of the label’s text. The below XPath uses the `<div>` tag to locate the `<label>` tag.

    XPath = //div[contains(@class,'checkbox']/label[.,'General updates']

The refined locator is much less fragile. It doesn’t break when the lengthy text changes. It only breaks when the @class attribute changes or the anchor text changes. But `@class` is the most important handle for styling web elements using virtually all CSS frameworks. Unless you change the whole front-end CSS framework, `@class` will stay the same most of the time.

## Tip #2. XPath axes are very handy sometimes
Let’s take a look at the [Your Cart](https://www.bestbuy.com/cart) page of the popular e-commerce site **Best Buy**. In this scenario, we’ll try to adjust the quantity of a certain cart item then check whether the total amount is calculated correctly. 

In the picture below, we’ve added **Apple Watch Series 4 (GPS + Cellular) - 40mm** to our cart. The unit price of this item is $499. If we buy 3 of them, we’d have to pay **$1,497**. Since every time people add an item to the cart, the default quantity is 1 (one), our automated test has to locate the **Quantity** dropdown list and change the value from 1 to 3 before doing the intended verification.

<a href="https://imgur.com/lHLniie"><img src="https://i.imgur.com/lHLniie.png" title="source: imgur.com" /></a>

The Quantity dropdown list we want to interact with is actually a `<select>` tag. Unfortunately, this `<select>` tag doesn’t have any unique attribute. The below picture shows the HTML makeup of this dropdown list.

<a href="https://imgur.com/UuvfemE"><img src="https://i.imgur.com/UuvfemE.png" title="source: imgur.com" /></a>

Its `@class` attribute (‘c-dropdown v-medium’) duplicates with other dropdown lists. On the other hand, the `@aria-label` attribute is too fragile. One small change to the lengthy product name will certainly break a bunch of tests so we’d highly recommend refraining from using locators similar to the below XPath. 

    XPath = //select[@aria-label='Apple – Apple Watch Series 4 (GPS + Cellular) 40mm Gold Aluminum Case with Pink Sand Sport Band – Gold Aluminum quantity']

The best way to locate this Quantity dropdown list is via a more easy-to-identify control: the `<a>` tag which contains the **skuId** that uniquely identifies this particular Apple Watch product regardless of how people name it. We can get this skuId either from the GUI or the database.

<a href="https://imgur.com/EV4idvu"><img src="https://i.imgur.com/EV4idvu.png" title="source: imgur.com" /></a>

Each `<a>` tag is paired with a `<select>` tag. Both elements reside in the same row which is a `<div>` tag. Thus, we can take advantage of the XPath axis “ancestor” to locate `<select>` with respect to `<a>`. The picture below describes the abstract DOM structure of our elements of interest.

<a href="https://imgur.com/NqlElWl"><img src="https://i.imgur.com/NqlElWl.png" title="source: imgur.com" /></a>

Based on the tactic above, the final XPath to locate our Quantity dropdown list looks like this:

    XPath = //a[contains(@href,'skuId=6139660')]/ancestor::div[@class='row']//select

This XPath might seem puzzling to some but in fact, it’s relatively simple. Imagine you’re a Test Automation engine being instructed to find the desired web element, here’s what the above XPath means:
* Find the `<a>` tag whose @href attribute contains the string ‘skuId=6139660’.
* Based on that `<a>` tag, find all of its ancestors (parent, grandparents, great grandparent, etc.).
* Among those ancestors, pick the one that is a `<div>` with the `@class` attribute equal to ‘row’.
* Based on that `<div>`, look for all tags inside of that `<div>` and pick out the `<select>` tag.
* Bingo. Since each row only contains one `<select>` tag, that `<select>` ought to be the Quantity dropdown list we’re looking for.

If you master the art of XPath axes, you possess a deadly weapon at your disposal. Learn more about XPath axes [here](https://www.w3schools.com/xml/xpath_axes.asp).

## Tip #3. Treat tables as tables
Modern front-end developers frown upon the `<table>` tag. And they have [good reasons](https://www.lifewire.com/dont-use-tables-for-layout-3468941) to do so. However, Homo sapiens still love presenting data in tabular format since the dawn of time. So what do they do? They nest `<div>` tags together to visually resemble a table, something like this:

    <div class="table">
      <div class="row">
        <div class="cell"></div>
        <div class="cell"></div>  
        <div class="cell"></div>
      </div>  
    </div>  

So “table” now becomes a concept completely unrelated to the physical `<table>` tag in HTML thanks to the magic of CSS styling. Although it’s good news for front-end developers and end-users (browsers tend to render `<div>` faster than `<table>`), it’s bad news for novice automation engineers. We got to understand the inner structure of these conceptual tables before we start interacting with them.

For instance, consider the below round-trip flight booking table on [Agoda](https://flights.agoda.com/). It shows you the prices with respect to your depart date (row) and return date (column). The highlighted cell tells us that if we want to fly from London to Paris on Saturday Feb 2nd and return on Wednesday Feb 6th, we got to pay $106.

<a href="https://imgur.com/xSNfU6q"><img src="https://i.imgur.com/xSNfU6q.png" title="source: imgur.com" /></a>

Our automated test will go through this table, picking a predefined cell, get its value (e.g. $106) then check the calculated amount that travelers have to pay after tax and services fees. To perform this test, we need to capture the [5, 5] cell. The HTML makeup of the desired cell looks like below.

<a href="https://imgur.com/Pb03Hx3"><img src="https://i.imgur.com/Pb03Hx3.png" title="source: imgur.com" /></a>

A locator based on the text value of this `<a>` tag is obviously unreliable since the price of air tickets fluctuate over time. It’s a bad practice to construct a locator for the target cell like below.

    XPath = (//div[@id='cTgI']//a[.='$106'])[13]

This XPath tells the automation engine to look for the 13th `<a>` tag whose inner text is $106. Our tests will certainly break if the price of this particular flight changes. Or even worse, this XPath also fails if any of the 12 cells prior to the desired cell changes its value.

To fix it, we’ll need to switch to a CSS selector as below. This CSS selector works well since it locates the cell by the values of its row and column (depart and return dates). You can replace the actual dates by two variables to make the locators more robust. 

    CSS = [data-y-filter-code='20190202'][data-x-filter-code='20190206']

Alternatively, if we insist on using XPath, we can locate the desired cell by its indexes. Note that our indexing excludes the header row and the header column. Therefore, our cell of interest resides in row `[5]` and column `[5]`. The picture below shows the HTML markup of the flight-booking table.

<a href="https://imgur.com/sMlLqjr"><img src="https://i.imgur.com/sMlLqjr.png" title="source: imgur.com" /></a>

We’d recommend the below locator since you can replace the indexes by two variables and reuse this one locator for any other cells in the same table.

    XPath = //div[contains(@class,'FilterMatrixTable')]//div[@class='keel-grid row'][5]//div[contains(@class,'col-cell')][5]/a

Note that we use `@class` to identify the table since its ID is dynamic (e.g. jb01). That means the ID changes every time you refresh the page. 

## Tip #4. Avoid absolute paths at all cost
An absolute XPath looks like this: 

    XPath = /html/body/div[2]/div/div[4]/div/div/div/div/div/section/div/div/
    article/div/form/div/div[7]/div/div[1]/label

Instead of double-slash (“//”) which denotes a relative query, the author of this XPath uses single-slash (“/”). That means each node in the chain must be a direct child of its parent. Needless to say, direct-linking like this imposes unnecessary rigidity on our query, making it super long and error-prone. So whenever you spot a lengthy XPath with a lot of single-slashes, you can tell right away that the XPath is bad.

## Tip #5. Always test your XPath
As a tester, we must test everything we do. Locators are no exception. Make sure that you have a way to ensure that a locator can uniquely identify one and only one web element. If a locator retrieves 2 or more elements, it’s not a good locator. Pro tip: you can take advantage of the Locator Test feature of [POM Builder](https://www.pombuilder.com) to make this task easier.

## Conclusion
Capturing a good locator that uniquely identifies a web element is truly an art. It requires a good instinct to distinguish the good from the bad, the reliable from the fragile. The above tips should be helpful to those who’ve just started their test automation journey. However, like almost every other art, you need to practice intensively to become a master. It takes serious time and effort.

To cut down the time and effort, you can leverage free browser extensions on Chrome Web Store such as [POM Builder](https://www.pombuilder.com). This tool helps you cut down the locator building time by half using a smart pattern-recognition algorithm. And the best part: it’s free.

If you stumble upon difficult situations in which you cannot find an adequate locator, feel free to let us know in the Comment section or email us at <pombuilder@logigear.com>. 

## About the Author
<table style="background:#edf0f4">
<tr>
    <td> <a href="https://imgur.com/xSFo9Ts"><img src="https://i.imgur.com/xSFo9Ts.png" title="source: imgur.com" style="width:400px;" /></a> 
    </td>
    <td>
    <p> <a href="https://www.logigear.com/magazine/author/Thuc-Nguyen/"><b>Thuc Nguyen</b></a> is Associate Product Manager of LogiGear’s automation solutions. Thuc has a great passion for product management, UX design, Blockchain, growth hacking and especially complex test automation problems. </p>
    </td>
</tr>
</table>