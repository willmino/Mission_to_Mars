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

Below is the parental `find_all()` function that has all the individual news article boxes. Each news article box has a title, link, and summary preview text among others.

`all_divs = soup.find_all('div', class_ = 'col-md-8')`

Here are the test extractions for all news article titles. I iterated through the article to extract the titles exclusively from the preview text to highlight the printed results. For the article `title` the `find()` function looked for all elements that contained the article titles on the webpage. These `div` elements were passed through the `find()` function along with the `class_` `content_title`. `.text` was chained to this function.
For the `preview` extraction, I passed the tag `div` and the `class_` `article_teaser_body` into the `find()` function.

`for div in all_divs:`

&nbsp;&nbsp;&nbsp;&nbsp;`title= div.find('div', class_ = 'content_title').text`

&nbsp;&nbsp;&nbsp;&nbsp;`print(title)`

![Article_titles](https://github.com/willmino/Mission_to_Mars/blob/main/images/article_titles.png)



`for div in all_divs:`

&nbsp;&nbsp;&nbsp;&nbsp;`preview = div.find('div', class_='article_teaser_body').text`

&nbsp;&nbsp;&nbsp;&nbsp;`print(preview)`

![Article_preview_text](https://github.com/willmino/Mission_to_Mars/blob/main/images/article_previews.png)

Once I confirmed the code successfully extracted the information, I began writing the Splinter automated script to extract the website information.

I first created a list called `mars_news_articles = []` which would hold a list of dictionaries. Each dictionary corresponded to a news article's title and the preview text. The keys for the dictionary were `title` and `preview`. At the end of the script, the list would hold all of the dictionaries containing the article information from the website. Below is the script.

`mars_news_articles = []` 

`for div in all_divs:`

&nbsp;&nbsp;&nbsp;&nbsp;`title = div.find('div', class_='content_title').text`

&nbsp;&nbsp;&nbsp;&nbsp;`preview=div.find('div', class_='article_teaser_body').text`

&nbsp;&nbsp;&nbsp;&nbsp;`mars_news_dict={}`

&nbsp;&nbsp;&nbsp;&nbsp;`mars_news_dict["title"] = title`

&nbsp;&nbsp;&nbsp;&nbsp;`mars_news_dict["preview"] = preview`

&nbsp;&nbsp;&nbsp;&nbsp;`mars_news_articles.append(mars_news_dict)`

To confirm the results of the script run, I printed the `mars_news_articles` list.

![mars_news_article_list](https://github.com/willmino/Mission_to_Mars/blob/main/images/mars_news_article_list.png)

After quitting the browser using `browser.quit()`, I stored the results of the automated web scrape into a JSON file and also to a Mongodb Database. The resulting JSON file can be found here.

[mars_new.json](https://github.com/willmino/Mission_to_Mars/blob/main/mars_news.json)

`mars_news_json = json.dumps(mars_news_articles)`

`with open("mars_news.json","w") as outfile:`

&nbsp;&nbsp;&nbsp;&nbsp;`outfile.write(mars_news_json)`







