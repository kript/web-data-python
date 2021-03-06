---
layout: page
title: Working With Data on the Web
subtitle: Getting Data
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Write Python programs to download data sets using simple REST APIs.

A growing number of organizations make data sets available on the web in a style called [REST](reference.html#rest),
which stands for REpresentational State Transfer.
The details (and ideology) aren't important;
what matters is that when REST is used,
every data set is identified by a URL.

For this example we'll use data generated by 15 global circulation models
that is provided through the World Bank's [Climate Data API](http://data.worldbank.org/developers/climate-data-api).
According to the API's home page,
the data sets containing yearly averages for various values are identified by URLs of the form:

<code>http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/<em>var</em>/year/<em>iso3</em>.<em>ext</em></code>

where:

*   *var* is either `pr` (for precipitation) or `tas` (for "temperature at surface");
*   *iso3* is the [International Standards Organization (ISO) 3-letter code for a specific country](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3),
    such as "CAN" for Canada or "BRA" for Brazil;
    and
*   *ext* (short for "extension") specifies the format we want the data in.
    There are several choices for format,
    but the simplest is [comma-separated values](reference.html#csv) (CSV),
    in which each record is a row,
    and the values in each row are separated by commas.
    (CSV is frequently used for spreadsheet data.)

For example, if we want the average annual temperature in Canada as a CSV file, the URL is:

~~~
http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv
~~~

If we paste that URL into a browser, it displays:

~~~
year,data
1901,-7.67241907119751
1902,-7.862711429595947
1903,-7.910782814025879
...
2007,-6.819293975830078
2008,-7.2008957862854
2009,-6.997011661529541
~~~

> ## Behind the Scenes {.callout}
> This particular data set might be stored in a file on the server, or the server might do this:
>
> 1.  Receive our URL.
> 2.  Break it into pieces.
> 3.  Extract the three key fields (the variable, the country code, and the desired format).
> 4.  Fetch the desired data from a database.
> 5.  Format the data as CSV.
> 6.  Send that to our browser.
>
> As long as the World Bank doesn't change its URLs,
> it can switch back and forth between these approaches without breaking our programs.

If we only wanted to look at data for two or three countries,
we could just download those files one by one.
But we want to compare data for many different pairs of countries,
which means we should write a program.

Python has a library called `urllib2` for working with URLs.
It is clumsy to use, though, so many people (including us) prefer
a third-party library called [Requests](http://docs.python-requests.org).
To install it, run the command:

~~~ {.bash}
pip install requests
~~~
~~~ {.output}
Requirement already satisfied (use --upgrade to upgrade): requests in /Users/gwilson/anaconda/lib/python2.7/site-packages
Cleaning up...
~~~

We get this message because we already have it installed;
if you don't, you'll see a different message.
We can now get the data we want like this:

~~~ {.python}
import requests
url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv'
response = requests.get(url)
if response.status_code != 200:
    print 'Failed to get data:', response.status_code
else:
    print 'First 100 characters of data are'
    print response.text[:100]
~~~
~~~ {.output}
First 100 characters of data are
year,data
1901,-7.67241907119751
1902,-7.862711429595947
1903,-7.910782814025879
1904,-8.15572929382
~~~

The first line imports the `requests` library.
The second defines the URL for the data we want;
we could just pass this URL as an argument to the `requests.get` call on the third line,
but assigning it to a variable makes it easier to find.

`requests.
et` actually gets our data. More specifically, it:

*   creates a connection to the `climatedataapi.worldbank.org` server;
*   sends it the URL `/climateweb/rest/v1/country/cru/tas/year/CAN.csv`;
*   creates an object in memory on our computer to hold the response;
*   assigns a number to the object's `status_code` member variable to tell us whether the request succeeded or not; and
*   assigns the data sent back by the web server to the object's `text` member variable.

The server can return many different [status codes](reference.html#http-status-code);
the most common are:

|Code|Name                 |Meaning                                                                   |
|----|---------------------|--------------------------------------------------------------------------|
|200 |OK                   |The request has succeeded.                                                |
|204 |No Content           |The server has completed the request, but doesn't need to return any data.|
|400 |Bad Request          |The request is badly formatted.                                           |
|401 |Unauthorized         |The request requires authentication.                                      |
|404 |Not Found            |The requested resource could not be found.                                |
|408 |Timeout              |The server gave up waiting for the client.                                |
|418 |I'm a teapot         |No, really...                                                             |
|500 |Internal Server Error|An error occurred in the server.                                          |

Of these, 200 is the one we really care about:
if we get anything else, the response probably doesn't contain actual data
(though it might contain an error message).

> ## Some People Don't Follow the Rules {.callout}
>
> Unfortunately, some sites don't return a meaningful status code.
> Instead, they return 200 for *everything*,
> then put an error message (if appropriate) in the text of the response.
> This works when the result is being displayed to a human being,
> but fails miserably when the "reader" is a program that can't actually read.

> ## How Hot is Afghanistan? {.challenge}
> 
> Read the [documentation](http://data.worldbank.org/developers/climate-data-api) for the Climate Data API,
> and then write URLs to find the annual average temperature for Afghanistan between 1980 and 1999.

> ## Defining REST API {.challenge}
>
> An REST API is:
> a) in CSV format
> b) a way of accessing data via an URL
> c) less work for the server
> d) only accessable via python libraries (e.g. requests)
>

