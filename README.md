# The Most Popular Events in Nashville
#### Based on data scraped from www.Nashville-Theatre.com

##Table of Contets
- [Motivation](#Motivation)
- [Data Questions](#Data-Questions)
- [Data Sources](#Technologies)
- [Process](#Process)
- [Dashboard](#Link-to-Dashboard)

## Motivation
I love attending concerts, musicals, stand-up comedy, and other types of events. Over the years I have read articles about how the 'Music City' is not just a great place to see a band, but a city full of other types of entertainment. I wanted to see if I could quantify the popularity of the different types of entertainment events in the Nashville area and challenge the notion of music being the most popular type of event in the 'Music City'.

## Data Questions
1. What types events in Nashville are most popular?
2. What venues book the most shows?
3. Of the most popular shows, what is the ticket capacity per show?
4. How do different types of shows compare? (e.g. How does a single sold out show at Bridgestone Arena compare to two weeks of sold out musical performances [of the same performance])

## Data Sources

## Technologies

## Process
Using requests, BeautifulSoup, and pandas in python, I established a web scraping connection between my Jupyter Notebook and Nashville-Theatre.com.

I discovered that the website has a number of different types of URL formats that made scraping the various levels of data a challenge. I couldn't simply scrape all categories or all events from a certain venue without using a number of different links. The website does not contain all shows in one page, and the most comprehensive layer of data is accessible on a month to month basis. This brought on the first break-through discovery in my code.


<details>
  <summary>*User-Defined Function* with a Loop that cycles through the months of the year.</summary>

```
def buildurl(item):
    try :
        return 'https://www.nashville-theatre.com/theaters' + item.find('h3').find('a')['href']
    except :
        return None

def getShows(month):
    yearlist = []
    New_URL = f'https://www.nashville-theatre.com/category_dates.php?year=2022&month={month}'
    r = requests.get(New_URL, headers=headers)
    nashville_soup = BS(r.text,'html.parser')
    shows = nashville_soup.find_all('div', {'class':'sbl2-heading'})
    for item in shows:
        show = {
        'event': item.find('h3').text,
        'venue': item.find('span',{'class':'sbl2-venue'}).text,
        'dates' : item.find('span',{'class':'sbl2-date'}).text,
        'link' : buildurl(item)
        }
        yearlist.append(show)
    return yearlist
```

</details>


I chose to analyze the most popular events between September 1st and December 31st, 2022 and used a loop to call my defined-function a created a DataFrame of unique events, their venue, range of dates showing, and if available a link to ticketing information.

### add image of dataframe
