# Background:
A web crawler browses through the entire or a specific subset of the Internet in order to index its content. Seeds are a set of URLs that act as the starting point for the crawler. For the sake of this problem, let’s assume that the seeds are RSS feed URLs of multiple news and blog publications. Since there are articles being published throughout the day, the RSS feeds are updated frequently. Some feeds are updated more often than others depending on feed size, article frequency, time of day, etc. 

  
It is very important for web crawlers to re-crawl the same feed repeatedly in order to capture its updates with minimum latency, but it’s also important to conserve bandwidth and processing time on rarely updated feeds. Determining re-crawling strategies has been a subject of active research in the web community.  

  
In this problem, we attempt to arrive at an optimal crawling strategy for a set of sites, balancing article freshness with crawl count and avoiding missed articles.

  

# Problem: 
In this problem, we simulate an efficient feed reader. Given historical information about article timestamps in each feed, you should schedule optimal times to poll over the next day (in UTC). 

  
Your program will be provided the ‘Feed ID’ and it should make a call to our simulated feed API with a poll time. The API will return the 10 latest article timestamps before that poll time which your program should use to schedule the next poll. If more than 10 articles have been published since your previous poll, the additional articles will not be captured!

  
This process should be repeated until the end of the day at which your program should print out the feed ids and times chosen for polls during that day. For development purposes, if you want to see the score for your polling strategy you can hit our scoring API endpoint(with appropriate data as described below in the Scoring API section) which will return you the score as well as the components of your score. 

  
Note: All timestamps that are referred in this problem are of the ISO-8601 format, in the UTC timezone.

  

# Dataset:
You will also be provided a historical training dataset of all article times for the set of sites over `2014-07-03 - 2014-07-24`. This can help if you wish to use optimization algorithms or machine learning/AI approaches to approach this problem.

  
The training dataset will have N rows. Each row has a Feed ID and the timestamps of articles for that site. Feed and Scoring API use the same Feed IDs as provided in this dataset. 


# Input:
```
<Feed API Endpoint>
<Date of Poll>
<Feed ID> 
<Feed ID> 
<Feed ID> 
…
```

  
The first row of the input file is the Feed API Endpoint, the second row is the date to poll and  N rows following that contain Feed IDs. We've provided two sample inputs for you to verify that your program responds to our input format.

  
For development purposes, you can use http://codejam.airpr.com/poll as the Feed API Endpoint and http://codejam.airpr.com/score as the Scoring API Endpoint.  This development API will only work for the first 100 Feed IDs in the provided training dataset, but will also access an additional day of feed data outside of the training set (`2014-07-25`). For ranking, you’ll be scored on the performance on the full set of Feed IDs for `2014-07-25`. 



# Output:
Should be a file with N rows. Each row should have the Feed ID and a comma separated sequence of timestamps to run polls on the date given (2nd row of the input file).


# Submitting:
Email codejam@airpr.com with a zip file containing your program file(s) and a build script to compile your code if necessary. To run your code, we will first call `./build.sh` and then `cat inputfile | ./codejam > outputfile`. 

We will execute your code on Ubuntu 14.04

Example layout:
  ./README.txt
  ./build.sh
  ./codejam
  ./models/mymodel.dat

# Feed API:

## Request:
```
GET <Feed API Endpoint>?feed_id=<Feed ID>&poll_time=<ISO 8601 formatted timestamp>
```
  

## Response:
```
{

  “feed_id”: <Feed ID>,

  “poll_time”: <ISO 8601 formatted timestamp>,

  “article_times: [<ISO 8601 formatted timestamp>, <ISO 8601 formatted timestamp>, <ISO 8601 formatted timestamp>... ]

}
```


# Scoring API:

## Request:
```
POST <Scoring API Endpoint>

with x-www-form-urlencoded parameters:

feed_id=<Feed ID>

poll_times[]=<ISO 8601 formatted timestamp>

poll_times[]=<ISO 8601 formatted timestamp>

poll_times[]=<ISO 8601 formatted timestamp>

...
```

## Response:
```
{

  “feed_id”: <Feed ID>,

“poll_times: [<ISO 8601 formatted timestamp>, <ISO 8601 formatted timestamp>, <ISO 8601 formatted timestamp>... ]

  “total_score”: Float,

  “freshness_score”: Float,

  “missed_article_penalty”: Float,

  “efficiency_bonus”: Float

}
```
  

# Examples:
Our test dataset for this example has 24 articles on 2015-08-02, one every hour.

  
Polling at 2015-08-02T23:59:59.000Z gives the latest 10 articles:

```
curl -X GET -H Cache-Control:no-cache <Feed API Endpoint>?feed_id=1&poll_time=2015-08-02T23:59:59.000Z

  
{

    "feed_id": 1,

    "poll_time": "2015-08-02T23:59:59.000Z",

    "article_times": [

        "2015-08-02T23:00:00.000Z",

        "2015-08-02T22:00:00.000Z",

        "2015-08-02T21:00:00.000Z",

        "2015-08-02T20:00:00.000Z",

        "2015-08-02T19:00:00.000Z",

        "2015-08-02T18:00:00.000Z",

        "2015-08-02T17:00:00.000Z",

        "2015-08-02T16:00:00.000Z",

        "2015-08-02T15:00:00.000Z",

        "2015-08-02T14:00:00.000Z"

    ]

}
```
  
Scoring this polling strategy of only one poll at the end of the day reveals 14 articles missed and low freshness score, although it is efficient.

```
curl -X POST -H Cache-Control:no-cache -H Content-Type:application/x-www-form-urlencoded -d 'feed_id=1&poll_times%5B%5D=2015-08-01T23%3A59%3A59' <Scoring API Endpoint>

  
{

    "feed_id": 1,

    "poll_times": [

        "2015-08-01T23:59:59.000Z"

    ],

    "total_score": "-1343.00",

    "freshness_score": "10.00",

    "missed_article_penalty": "-1400.00",

    "efficiency_bonus": "47.00"

}
```
  
Polling more often avoids the missed article penalty, although this can still be improved:

```
curl -X POST -H Cache-Control:no-cache -H Content-Type:application/x-www-form-urlencoded -d 'feed_id=1&poll_times%5B%5D=2015-08-01T00%3A00%3A59&poll_times%5B%5D=2015-08-01T04%3A00%3A59&poll_times%5B%5D=2015-08-01T08%3A00%3A59&poll_times%5B%5D=2015-08-01T12%3A00%3A59&poll_times%5B%5D=2015-08-01T16%3A00%3A59&poll_times%5B%5D=2015-08-01T20%3A00%3A59&poll_times%5B%5D=2015-08-01T23%3A00%3A59' <Scoring API Endpoint>

  
  
{

    "feed_id": 1,

    "poll_times": [

        "2015-08-01T23:00:59.000Z",

        "2015-08-01T20:00:59.000Z",

        "2015-08-01T16:00:59.000Z",

        "2015-08-01T12:00:59.000Z",

        "2015-08-01T08:00:59.000Z",

        "2015-08-01T04:00:59.000Z",

        "2015-08-01T00:00:59.000Z"

    ],

    "total_score": "159.56",

    "article_score": "118.56",

    "missed_article_penalty": "0.00",

    "efficiency_bonus": "41.00"

}
```

# Questions:
Any issues or questions? Open an issue on GitHub! https://github.com/airpr/airpr-code-jam/issues
