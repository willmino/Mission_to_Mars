# Mission_to_Mars
App for automated webscraping of data from the NASA Mars Mission website using python, BeautifulSoup, and Splinter

## Purpose

I generated a report of Mars data in Python, using Pandas, Splinter and BeatifulSoup to automate web scraping of HTML code and data storage from the NASA Mars Planet Science website. In my first script, I extracted the article title and a brief summary description from each article on the website. I also used Splinter to scrape Mars temperature data from the website's HTML table. The scraped data was stored in a Pandas dataframe and exported to a Mongo database and CSV file. I then analyzed the data using Pandas and matplotlib to generate weather statistics like the average monthly low temperature and pressure on Mars.

## Sources

[Mars News](https://redplanetscience.com)

[Mars Temperature Data Website](https://data-class-mars-challenge.s3.amazonaws.com/Mars/index.html)

# Analysis

### Deliverable 1

Splinter and BeautifulSoup Dependencies were imported to obtain the automated web browser functionality in python and HTML code parsing, respectively.

`from splinter import Browser`

`from bs4 import BeautifulSoup as soup`

`from webdriver_manager.chrome import ChromeDriverManager`

An executable path of Google ChromeDriver was installed in this script. Then, an instance of the browser was opened using Google Chrome.

`executable_path = {'executable_path': ChromeDriverManager().install()}`

`browser = Browser('chrome', **executable_path, headless=False)`

The browser visited the first website using the below code.

`url = 'https://redplanetscience.com'`

`browser.visit(url)`

The HTML code was obtained from the website and BeautifulSoup parsed the HTML code.

`html = browser.html`

`soup = soup(html, 'html.parser')`

A test extraction of the HTML elements was performed. We first designated the variable `all_divs` set equal to the `find()` function, passing `div` tag and `col-md-8` class designations throught it, all chained to the `soup` parsed HTML code from this website.


`# Scrape all the article titles and preview text from the NASA mars news website`

Below is the parental `find_all()` function that has all the individual news article boxes. Each news article box had a title, link, and summary preview text among others.

`all_divs = soup.find_all('div', class_ = 'col-md-8')`

Here are the test extractions for all news article titles. I iterated through the article to extract the titles exclusively from the preview text to highlight the printed results. For the article `title` the `find()` function looked for all elements that contained the article titles on the webpage. These `div` elements were passed through the `find()` function along with the `class_` `content_title`. `.text` was chained to this function to obtain the article title text.
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

[mars_news.json](https://github.com/willmino/Mission_to_Mars/blob/main/mars_news.json)

`mars_news_json = json.dumps(mars_news_articles)`

`with open("mars_news.json","w") as outfile:`

&nbsp;&nbsp;&nbsp;&nbsp;`outfile.write(mars_news_json)`

The below code was used in my computer's command terminal to export the JSON file into mongodb.

`from pymongo import MongoClient`

`mongo = MongoClient(port=27017)`

`mongoimport --type json -d mars_mission -c mars_news --drop --jsonArray mars_news.json`

![MongoDB_Mars_News](https://github.com/willmino/Mission_to_Mars/blob/main/images/mongodb_mars_news.png)

### Deliverable 2

A second automated Splinter script was created to scrape Mars temperature data from an HTML table on the NASA Mars website. For this script we imported the following dependencies.

`from splinter import Browser`

`from bs4 import BeautifulSoup as soup`

`from webdriver_manager.chrome import ChromeDriverManager`

`import matplotlib.pyplot as plt`

`import pandas as pd`

The script downloaded the Google ChromeDriver for use of Chrome in Splinter. I created a `browser` instance using Splinter and passed `'chrome'` through the `Browser` function. Running the below code opened a Google Chrome browser.

`executable_path = {'executable_path': ChromeDriverManager().install()}`

`browser = Browser('chrome', **executable_path, headless=False)`

I visited the Mars temperature data website using the code below.

`url = 'https://data-class-mars-challenge.s3.amazonaws.com/Mars/index.html'`

`browser.visit(url)`

The HTML code was obtained from the website using `html = browser.html` and the code was parsed using `BeautifulSoup` as `soup`.

`html = browser.html`

`soup = soup(html, 'html.parser')`

To extract all the data from the HTML table, I first formulated a plan. Before setting up the plan, I needed to decipher the code structure of the HTML table by viewing the HTML code using Chrome's DevTools within the browser. The HTML table followed this hieracy of HTML elements: `<table />`,`<tbody />`,`<tr />`, `<td />`. From left to right, the most parental element (all inclusive element) contains the subsequent elements. The `<td />` table data element was nested in the `<tr />` element, which itself was nested in the `<tbody />` element and this was in turn contained in the `<table />` element. 

Once I established the structure of the HTML table, I began to formulate my plan to extract the data. I decided to iterate through all of the `<tr />` table row elements. I would then look at all of the `<td />` table data elements within the table rows and create a list of all the `<td />` elements for each `<tr />` element. Then, I added the list of each `<tr />` element to a list of lists. Each index position in this list of lists represented the list of table data for each row.

To execute this plan, I created the variable `all_trs` and set it equal to `soup` with the `find_all()` function chained to it. The parameters passed through this function were the `tr` element (table row) and the `class_` called `data-row`. This allowed the BeautifulSoup code to capture the HTML data for each container for the rows of data. I also created the `list_of_lists` list to hold each row of data from the table.

`all_trs = soup.find_all('tr', class_='data-row')`

`list_of_lists = []`

I needed to iterate through each row, create a list for each row, and look for all the `<td />` elements in each row. To capture all `<td />` elements per row, I set the variable `row` equal to each `<tr />` element with the `find_all()` function chained to it, and I passed the `<td />` element through the function. Once I gained functionality to iterate through each row, I then made the script iterate through each `<td />` element within each row. To capture each `<td />` elements text, I had set the variable `each_column_in_row` equal to each `<td />` element with the `.text` attribute chained to it. Then, I appended the text from each `<td />` element to the `row_list` which would contain a list of all  table column `<td />` elements for each row once the code finished iterating through each row. After the next `for` loop was carried out for each row of data, I then appended each entire row of data to the `list_of_lists` list which would serve as the intermediate storage source for the entire table of data. The code block below summarized these functions.

`for tr in all_trs:`

&nbsp;&nbsp;&nbsp;&nbsp;`row_list = []`

&nbsp;&nbsp;&nbsp;&nbsp;`row = tr.find_all('td')`

&nbsp;&nbsp;&nbsp;&nbsp;`for td in row:`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`each_column_in_row = td.text`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`row_list.append(each_column_in_row)`

&nbsp;&nbsp;&nbsp;&nbsp;`list_of_lists.append(row_list)`

I also pythonically obtained the column headers for the table by carrying out a similar function used for the table data extraction. However, this function was simpler and only iterated through one row of data. That single row of data contained all column headers as opposed to the data points per column we wanted to extract from the `<td />` element. I took the initial `soup` parsed HTML code from the website and chained the `find()` function to it and passed the `th` table header element through the function. This would find and store the first `<th />` element in the `column_headers` variable. Then, I set the `column_names` variable equal to the the `column_headers` variable but I chained the `find_all()` function to it and this was designated to return all of the `<th />` elements within the first `<th />` element. The code below describes how this was accomplished and it highlights the method for obtaining the text from each column header and that each column header was added to list called `column_name_list`. 

`column_headers = soup.find('tr')`

`column_names = column_headers.find_all('th')`

`column_name_list = []`
`for name in column_names:`
    `column_name = name.text`
    `column_name_list.append(column_name)`

`print(column_name_list)`

The column name list returned was :

`['id', 'terrestrial_date', 'sol', 'ls', 'month', 'min_temp', 'pressure']`.

Finally, I stored the table data in a dataframe by using the code:

`mars_df = pd.DataFrame(list_of_lists , columns = column_name_list)`.

A sample of the resulting dataframe was included below.

![mars_df](https://github.com/willmino/Mission_to_Mars/blob/main/images/mars_df.png)

I then used the dataframe to answer some important analysis questions for the Mars Data report.

## Results

From viewing the resulting appropriately converted mars_df dataframe with all the data from the website's HTML table code, I could begin addressing some key findings for a report.

1. I found there are 12 months on the planet Mars. The line of code `mars_df["month"].max()` yielded the number 12. Once the highest month is reached on Mars, 12, the calendar on Mars resets back to the first month of the Martian year.

2. The website had about 1867 sols (Martian days) worth of data by using the code `mars_df["sol"].count()`. Even though there were higher sol day readings than the total count of rows obtained (ex: days greater than 1867), row data was absent for many sols. Thus, taking a count of the rows in the dataset was sufficient to reveal how many sols worth of data we had.

3. The coldest and warmest months on Mars were determined by this code:

`mean_min_temp_per_month = mars_df[["min_temp"]].groupby(mars_df["month"]).mean()`

I created a new dataframe, `mean_min_temp_per_month` that contained the `month` and `min_temp` columns from the original set. However, the temperature data was grouped by each month and the average low temperature for each month was obtained by using the `groupby()` function and the `.mean()` function. Since I created a new dataframe, it was easy to plot the average monthly low temperatures as a bar chart. This was accomplished using the below lines of code:

`plt.bar(mean_min_temp_per_month.index, mean_min_temp_per_month["min_temp"])`

`plt.xlabel("Martian Month")`

`plt.ylabel("Average Low Temperature (C)")`

`plt.title("Average Martian Low Temperature per Month")`

The resulting plot was displayed:

![Mean_Low_Per_Month](https://github.com/willmino/Mission_to_Mars/blob/main/images/mean_min_temp_per_month.png)

I could tell from the plot that the coldest month, with the lowest average temperature, was the 3rd month with an average low temperature of about -83.31 degrees. The warmest month on Mars was the 8th month with an average low temperature of -68.38 degrees.
The exact min and max values for the average monthly low temperatures were determined by these lines of code.

`print(mean_min_temp_per_month.min())`

`print(mean_min_temp_per_month.max())`

4. The Martian months with the lowest and highest average pressure readings were observed by plotting that data in another bar chart.

First, a new dataframe was created by retrieving the `pressure` and `month` columns from the mars_df dataframe. The `groupby()` function was chained to the `pressure` column and the `month` column was passed through the `groupby()` function. The `mean()` function was chained to the end of the code. This produced a dataframe that had each month along with its corresponding average monthly atmostpheric pressure readings.

`average_pressure_per_month = mars_df[["pressure"]].groupby(mars_df["month"]).mean()`

I then plotted this dataframe into a bar chart using the below code.

`plt.bar(average_pressure_per_month.index,average_pressure_per_month["pressure"])`

`plt.xlabel("Martian Month")`

`plt.ylabel("Average Atmospheric Pressure")`

`plt.title("Monthly Atmospheric Pressure on Mars")`

![Monthly_average_atmospheric_pressure](https://github.com/willmino/Mission_to_Mars/blob/main/images/month_mars_atmos_pressure.png)

To retrieve the exact values for the lowest and highest monthly atmospheric pressures, the following lines of code were executed:

`print(average_pressure_per_month.min())`

`print(average_pressure_per_month.max())`

This code determined that the lowest average monthly atmospheric pressure was 745.05 and this corresponded to the 6th Martian month, as viewed on the plot. The highest average monthly pressure was 931.31 and this corresponded to the 9th Martian month.

5. To determine the number of Earth days in a Martian year, I plotted the cycle of minimum temperature readings taken over the course of the mars mission, versus the Earth days on which the readings were taken. When viewing the cyclicle patterns exhibited by the low and high points of temperature readings, I could visually estimate how many Earth days were in a Martian year. To do this, I observed how many days elapsed between two neighboring peaks of temperature values.

The below block of code illustrated how the plot was constructed. The count for number of earth days was achieved by chaining the `count()` function to the `terrestrial_date` column of the `mars_df` dataframe. I then pythonically created an array that had a list consisting of each Earth day. The `earth_days` function was then set equal to the list comprehension `x for x in earth_day_array` which extracted all the array values for each Earth day into a list. With this list, I could finally plot the `number of earth days` on the x axis and the Martian `min_temp` (y axis) reading that was taken on each Earth day.

`import numpy as np`

`earth_day_count = mars_df["terrestrial_date"].count()`

`earth_day_array = np.arange(1,earth_day_count+1)`

`earth_days = [x for x in earth_day_array]`

`plt.plot(earth_days, mars_df["min_temp"])`

`plt.xlabel("Number of Terrestrial Days")`

`plt.ylabel("Average Minimum Martian Temperature")`

`plt.title("Earth Days vs Martian Annual Weather")`

![Earth_Days_versus_Mars_Weather](https://github.com/willmino/Mission_to_Mars/blob/main/images/Earth_days_vs_Mars_weather.png)

To visually estimate the number of Earth (terrestrial) days in a Martian year, I found two terrestrial days with peak temperature readings close to the cleanest intervals on the x axis. These terrestrial days were on day 1430 (almost half-way between 1250 and 1500) and day 750. The distance between either two extreme minima or two extreme maxima will help approximate the length of a cycle in Martian annual temperature and thus the length of a Martian year. I subtracted the terrestrial day values between two close peaks (maxima) to get this estimation: `1430-750 = 680`. Thus, I determined there is approximately 680 Earth days in a Martian year. There is exactly 687 Earth days in a Martian year according to the below website. Thus, my approximation is accurate. This was achieved with less than a 25% margin of error. 

[NASA Mars Article](https://mars.nasa.gov/resources/21392/mars-in-a-minute-how-long-is-a-year-on-mars/#:~:text=The%20Earth%20zips%20around%20the,year%20means%20longer%20seasons%20too.")

`100*(687 - 680)/(687) = 1.02% Margin of Error`

The final dataframe `mars_df` was exported as a csv file from the python code to the Mission_to_Mars github repository:

[mars_data.csv](https://github.com/willmino/Mission_to_Mars/blob/main/mars_data.csv)



















