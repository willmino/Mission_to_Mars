# Mission_to_Mars
App for automated webscraping of data from the NASA Mars Mission website using python, BeautifulSoup, and Splinter

## Purpose

I generated a report of Mars data in python, using Pandas, Splinter and BeatifulSoup to automate web scraping of HTML code and data storage from the NASA Mars Planet Science website for news article titles and summaries. I also used Splinter to scrape Mars temperature data from the website's HTML table. The scraped data was stored in a Pandas dataframe and exported to a Mongo database and CSV file. I then analyzed the data using Pandas and matplotlib to generate weather statistics like the average monthly low temperature and pressure on Mars

## Sources

[Mars News](https://redplanetscience.com)

[Mars Temperature Data Website](https://data-class-mars-challenge.s3.amazonaws.com/Mars/index.html)

# Analysis


Splinter and BeautifulSoup Dependencies were imported to obtain the automated web browser functionality in python and HTML code parsing, respectively.

`from splinter import Browser`

`from bs4 import BeautifulSoup as soup`

`from webdriver_manager.chrome import ChromeDriverManager`

An executable path of Google ChromeDriver was installed in this script. Then, an instance of the browser was opened using Google Chrome.

`executable_path = {'executable_path': ChromeDriverManager().install()}`

`browser = Browser('chrome', **executable_path, headless=False)`

We visited the first website using the below code.

`url = 'https://redplanetscience.com'`

`browser.visit(url)`

The HTML code was obtained from the website and BeautifulSoup parsed the HTML code.

`html = browser.html`

`soup = soup(html, 'html.parser')`

A test extraction of the HTML elements was performed. We first designated the variable `all_divs` set equal to the `find()` function, passing `div` tag and `col-md-8` class designations throught it, all chained to the `soup` parsed HTML code from this website.


`# Scrape all the article titles and preview text from the NASA mars news website`

Below is the parental `find_all()` function that has all the individual news article boxes. Each news article box has a title, link, and summary description among others.

`all_divs = soup.find_all('div', class_ = 'col-md-8')`

# scrape all the article titles from the NASA mars news website and print them to confirm the results before running the 

for div in all_divs:
    title= div.find('div', class_ = 'content_title').text
    print(title)
# scrape all the article preview text from the NASA mars news website

for div in all_divs:
    preview = div.find('div', class_='article_teaser_body').text
    print(preview)




