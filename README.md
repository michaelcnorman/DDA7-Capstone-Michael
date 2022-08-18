# Analyzing the Popularity of Events in Nashville
#### Based on events listed on Nashville-Theatre.com
A web-scraping data analysis project conducted in Python by Michael Norman to satisfy completion of the Full Time Data Analytics Bootcamp at Nashville Software School.

## Table of Contets
- [Motivation](#Motivation)
- [Data Questions](#Data-Questions)
- [Technologies](#Technologies)
- [Process](#Process)
- [Code](#Code)
- [Dashboard](#Link-to-Dashboard)
- [Data Sources](#Data-Sources)

## Motivation
Nashville, TN is known for its robust concert experience. With numerous stages at varying sizes,  a live band is easy to find any night of the week. The saturation of music in Nashville is high, and it might be tough to imagine what else you might do for fun aside from going to see a band. I wanted to challenge the notion of Nashville as the "Music City" and quantify the popularity of events.


## Data Questions
1. What events are taking place in Nashville the fall 2022?
2. What types of events are they?
3. What day of the weeks are events occurring most often?
4. Which type of event is most popular?
5. How does a single large capacity event compare to a smaller capacity event that books recurring events of the same show?

## Technologies

1. Python with Jupyter Notebook
2. Requests, BeautifulSoup, Pandas, Regular Expressions,
literal_eval
3. Scraped HTML and JavaScript
4. Tableau (dashboard available on Tableau Public). https://public.tableau.com/app/profile/michael.c.norman
5. Google Site used to present findings.


## Process
Using Requests, BeautifulSoup, Pandas, Regular Expressions, and literal_eval in python, I established a web scraping connection between my Jupyter Notebook and www.Nashville-Theatre.com.

I discovered that the website has a number of different types of URL formats that made scraping the various levels of data a challenge. There was not a single link that contained all information for a show. In many cases, two links were needed, but in order to also include categorical information, a third link was needed. The website does not contain all shows in one page, and the most comprehensive layer of data is accessible on a month to month basis. This brought on the first break-through discovery in my code.

To begin, I created a series of defined functions to build the various URL's of the website so that I could call them when necessary. I created *getShows*, which cerated my initial DataFrame containing all unique events, their venues, and a link to that shows ticketing information, and their categorical information link (if available). You can see in my examples how I used a for loop to *extend* a list of shows by passing the range 9 through 13 through the *getShows* function.

After I had all my events and venues I was able to start using that DataFrame as a launching pad to build all my other URL's. It was around this part of the puzzle that I learned that the venue syntax in URL's wasn't always the same. Some venues used a - to mark spaces in the name, some had no spaces, and all used lower case letters. I also realized that all of the event names were lower case and if a space was in the title a - was used. Fortunately, I learned that each venue was called the same way in the URL's regardless of the part of the website. So, to overcome overwriting the venue names in my DataFrame I created a dictionary of venue names and assigned the URL formatting to the keys. Since the syntax for event names was always formatted the same, I was able to simply use *replace()* and *lower()* to strip any special characters or spaces and replace them with the appropriate syntax.  

There were three main scrapes for this project.
1. Obtaining all unique events, their venues, and grabbing their appropriate ticketing and information links
2. A loop that took each ticketing link in the DataFrame I created and pulled the show dates and times for that event. This step was the heaviest scrape and contained HTML and JavaScript and required using list comprehension for each occurence of an event in order to ensure that dates, and showtimes appropriately lined up with their event and venue without unintentionally duplicating the data.
3. Using a similar loop to iterate through the infolink for each event that had one and pull its category tags.

## Code

<details>
  <summary>*Defined Functions and Loops Used for Scraping*</summary>

```
def buildurl(item):
    try :
        return 'https://www.nashville-theatre.com/theaters' + item.find('h3').find('a')['href']
    except :
        return None

def buildticketurl(item):
    try :
        return 'https://www.nashville-theatre.com' + item.find('a',{'class':'category-dperformance'})['href']
    except :
        return None

def buildinfourl(item):
    try :
        return 'https://www.nashville-theatre.com/theaters'+ item.find('h3').find('a')['href']
    except :
        return None

def ticketInfo(soup):
    try :
        dictionary = literal_eval(re.search('var ticketPerformances = ({.+})',soup.prettify()).group(1))
        return dictionary
    except :
        return None

def getShows(month):
  yearlist = []
  New_URL = f'https://www.nashville-theatre.com/category_dates.php?year=2022&month={month}'
  r = requests.get(New_URL, headers=headers)
  nashville_soup = BS(r.text,'html.parser')
  shows = nashville_soup.find_all('div', {'class':'showbloclisting-v2'})
  for item in shows:
      show = {
      'event': item.find('h3').text,
      'venue': item.find('span',{'class':'sbl2-venue'}).text,
      'ticketlink' : buildticketurl(item),
      'infolink': buildinfourl(item)
      }
      yearlist.append(show)
  return yearlist

showlist = []

for monthnum in range(9,13):
    showlist.extend(getShows(monthnum))

show_times = []

for index, row in unique_shows_df.iterrows():
    ticketing = row['ticketlink']
    event = row['event']
    venue = row['venue']
    if ticketing:
        tr = requests.get(ticketing, headers=headers)
        ticket_soup = BS(tr.text,'html.parser')
        dictionary = ticketInfo(ticket_soup)
        location = ticket_soup.find('div',{'class':'locationdesk'}).text
        if dictionary:
            eventlist = []
            venuelist = []
            datelist = []
            timeslist = []
            locationlist = []
            for date,soup in dictionary.items():
                soup = BS(soup)
                eventlist.extend([event for entry in soup.find_all('div',{'class':'fc-perf-time'})])
                venuelist.extend([venue for entry in soup.find_all('div',{'class':'fc-perf-time'})])
                locationlist.extend([location for entry in soup.find_all('div',{'class':'fc-perf-time'})])
                datelist.extend([date for entry in soup.find_all('div',{'class':'fc-perf-time'})])
                timeslist.extend([entry.text for entry in soup.find_all('div',{'class':'fc-perf-time'})])
                shows = {
                  'name' : eventlist,
                  'venue': venuelist,
                  'location' : locationlist,
                  'dates': datelist,
                  'times': timeslist
                     }
                show_times.append(shows)
categories = []

for index, row in names_and_venues.iterrows():
  event = row['event'].replace(':','').replace("lace(' ','-').lower()
  venue = row['venue']
  venue = venue_dict[venue]
  shows_url = f'https://www.nashville-theatre.com/theaters/{venue}/{event}.php'
  r = requests.get(shows_url, headers=headers)
  shows_soup = BS(r.text,'html.parser')
  tags = {
    'name' : [row['event']for entry in shows_soup.find_all('a',{'id':'top-nav-1'})],
    'venue': [row['venue']for entry in shows_soup.find_all('a',{'id':'sub-nav-1'})],
    'tags' : [entry.text for entry in shows_soup.find_all('a', {'clreverse'})]
        }
categories.append(tags)
```
  </details>

## Dashboard

https://public.tableau.com/app/profile/michael.c.norman

## Data Sources
  <details>
    <summary>*Links to Sources*</summary>

    https://www.Nashville-Theatre.com

    *Capacity of Events
    https://en.wikipedia.org/wiki/Bridgestone_Arena
https://www.tpac.org/rentals/james-k-polk-theater/
https://www.tpac.org/rentals/andrew-jackson-hall/
https://nashvilledowntown.com/go/ascend-amphitheater
https://www.firstbankamphitheater.com/
https://www.opry.com/story/a-look-at-the-6-homes-of-the-grand-ole-opry
https://ryman.com/about/
https://www.visitmusiccity.com/local-business/marathon-music-works-0
https://www.countrymusichalloffame.org/plan-your-visit/venue-rental/spaces/cma-theater
https://www.nashvilleauditorium.com/general-information
https://citywinery.com/nashville/Online/default.asp?BOparam::WScontent::loadArticle::permalink=nashville-venue-information&BOparam::WScontent::loadArticle::context_id=&menu_id=B0A4DA65-1FC7-4AFA-98C5-FD164575AFA1
https://www.tpac.org/rentals/war-memorial-auditorium/
https://www.nashvillesymphony.org/about/schermerhorn-symphony-center/
https://en.wikipedia.org/wiki/Zanies_Comedy_Club
https://www.3rdandlindsley.com/faq/
https://www.indieonthemove.com/venues/the-basement-east-nashville-tennessee
https://www.countrymusichalloffame.org/plan-your-visit/venue-rental/spaces/cma-theater
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwi8lqjzrbr5AhXtkmoFHSrYAuUQFnoECBoQAw&url=https%3A%2F%2Fwww.vanderbilt.edu%2Fstudentcenters%2Freservation-guidelines%2F&usg=AOvVaw3Rj2YhUyW2RvuLH2IEJy1C
https://exitin.com/booking/
https://www.indieonthemove.com/venues/the-end-nashville-tennessee
https://www.rymanhp.com/property/wildhorse-saloon/
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjT4-yCrrr5AhXOgmoFHXZ-CzwQFnoECC8QAQ&url=https%3A%2F%2Fwww.stadiumsofprofootball.com%2Fstadiums%2Fnissan-stadium%2F&usg=AOvVaw0etRSk76pt05BwM65zPyZ6
https://www.tpac.org/rentals/andrew-johnson-theater/
https://www.indieonthemove.com/venues/the-basement-nashville-tennessee
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjintPZs7r5AhUylmoFHX8YDJYQFnoECB4QAw&url=https%3A%2F%2Fwww.mercylounge.com%2Fabout%2F&usg=AOvVaw3hD9aOrBluh7_v7-N_Q_kN
  </details>
