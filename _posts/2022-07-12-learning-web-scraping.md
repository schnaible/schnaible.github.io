---
layout: post
title: "Learning How to Scrape the Web"
date: 2022-07-12 14:20:48 -0700
categories: jekyll update
---

### Introduction

I recently decided to learn about WebScraping and dive into automation with Python.
My plan was to eventually make a simple bot that can email me daily with things like the weather, Twitter feed, news, etc. It appears that Twitter is pretty strict with their API so we'll see if I'm able to obtain the elevated access. One common source that was recommended online frequently is [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/).

Another resource that I found was [Quotes to Scrape](http://quotes.toscrape.com/). This is the site that we will use to scrape in this mini-tutorial. I will assume you know the basics of Python and are comfortable setting up an environment to work in. I personally did this using [Google Colab](https://colab.research.google.com/).

If you want to view the complete code feel free to skip to the bottom of this page.

### Initial Steps

There is not much to do here except importing the python libraries that we need. For web scraping we need the requests library and [BeautifulSoup4](https://pypi.org/project/beautifulsoup4/). I also imported random, since I wanted to grab a random page number and quote.

    import random
    import requests
    from bs4 import BeautifulSoup

### Grabbing the Page

Here we set a variable `page` equal the the URL that we want to scrape. We can grab the site using `requests.get(URL)`

    page = requests.get("https://quotes.toscrape.com")

If we run `print(page)` we see the output `<Response [200]>`. This is the "OK" status code for HTTP Requests, meaning we have successfully connected to the website. If you see a different response, like a 404, the site you are trying to access may not be online.

### Viewing HTML Content

In order to view the HTML content of `page` we can run `print(page.content)`, but this is hard to read and contains a bunch of `\n` and other characters that we do not care about. Let's make it easier to read by parsing it with BeautifulSoup.

    soup = BeautifulSoup(page.content, 'html.parser')

Here we are assigning the variable `soup` to the parsed page content that is returned by BeautifulSoup. If we `print(soup)` we can immediately see that `soup` is a lot cleaner than `page.content`.

At This point we must use our knowledge of HTML and analyze where the content we want is located.

### Pulling Quotes

#### Selecting div

For this tutorial we want to access one of the quotes on the page. In order for us to find where the quote is located in the HTML content of `soup`, let's go to the page that we are scraping. On this page right click, then click Inspect. This will open up the inspector tool.

From here click on the 'select element' button in the top left (The symbol is a cursor surrounded by a box). Next hover your mouse over the quote, ensuring that the highlighted section contains the desired information. Once we click we see that the HTML content in the Inspector highlights the following line:

    <div class="quote" itemscope="" itemtype="http://schema.org/CreativeWork">

![Selecting a Quote Using Inspect Element](/assets/images/selectQuote.png "Selecting Quote")

By doing this we learn that the quotes are stored in a `div` where `class="quote"`. Back in our python program we can finally pull the quotes from the HTML Content.

    quotes = soup.find_all(class_="quote")

Once again we have a new variable. This time `quotes` will be storing all of the information within every instance of `<div class="quote">`.

#### Separating Text and Author from HTML

Right now `quotes` contains a list of all `div` that have the `class="qoute`. Let's focus on the first quote with the following:

    quote[0]

When looking at an individual quote we see that it contains several other `div`. We want to pull the text of the quote along with the author. To do this we will perform some steps similar to when we localized the `<div class="quote">`. The following two lines contain the information we want:

    <span class="text" itemprop="text">“Anyone who has never made a mistake has never tried anything new.”</span>
    <span>by <small class="author" itemprop="author">Albert Einstein</small>

We have two new classes to localize. `<span class="text">` for the text and `<small class="author">` for the author. Focusing on just the text for now, let's use the `find_all()` method again.

    quote[0].find_all(class_="text")

Now we are just looking at the single line containing the text. If we run the line above we see that it is encompassed in a pair of square brackets `[]`. This means it is a single item in a list. Removing it from a list is as simple as:

    quotes[0].find_all(class_="text")[0]

Lastly, all we need to do is separate the text from the HTML tags. This is another simple addition to the line above. All we need to do is add `.text` to the end. We will also assign it to the variable `quote`.

    quote = quotes[0].find_all(class_="text")[0].text

To get the author we can do the exact same thing, but this time we will use `find_all(class_="author")`. We will store this information in the variable `author`.

    author = quotes[0].find_all(class_="author")[0].text

### Finishing Up

That's it!!

We have the quote and author separated in to their own variables, `quote` and `author`. With this we can print the quote with a simple print.

    print(quote + '\n' + author)

Below is a complete program to generate a random quote. Also available on my [GitHub](https://github.com/schnaible/WebScrapeBot).

    import random
    import requests
    from bs4 import BeautifulSoup

    # The page we want to scrape. Tests connection
    pageNum = random.randint(1, 10)
    page = requests.get("https://quotes.toscrape.com/page/" + str(pageNum))
    print(page.url)

    # Get the HTML content and find all instances where class is quote
    soup = BeautifulSoup(page.content, 'html.parser')
    quotes = soup.find_all(class_="quote")

    # Each page has 10 quotes, generate a random value in this range and pull the quote
    randInt = random.randint(0, 9)

    quoteText = quotes[randInt].find_all(class_="text")[0].text     # Get the quote from the inner class
    quoteAuthor = quotes[randInt].find_all(class_="author")[0].text # Get the Author from the inner class

    print(quoteText + '\n' + quoteAuthor)
