# News Scraper

News Scraper is asyncronous python program that scrapes news headlines using Splash.

Run ```http://localhost:8080/news``` in your browser after you have finished installing dependencies

## Prerequisite

- Docker: we need Docker containers to be able to run the following

```docker pull scrapinghub/splash```
```docker run --rm -p 8050:8050 scrapinghub/splash```

- Python: we need Python software to be able to run our code/program

- Pip: we need pip to be able to install our program dependencies. Pip comes with Python Software

## Dependencies

- Docker: download Docker for your machine

- Python: download python for your machine

- Run ```pip install requirements.txt``` to install all the requirements

## Case Study: Scraping the News

aiohttp can be used both as a server and a client library, like the very popular (but
blocking!) requests library. I wanted to showcase aiohttp by using an example that
incorporates both features.
In this case study, we’ll implement a website that does web scraping behind the
scenes. The application will scrape two news websites and combine the headlines into
one page of results. Here is the strategy:

1. A browser client makes a web request to <http://localhost:8080/news>.
2. Our web server receives the request, and then on the backend fetches HTML data
from multiple news websites.
3. Each page’s data is scraped for headlines and content.
4. The headlines are sorted and formatted into the response HTML that we send
back to the browser client.

Web scraping has become quite difficult nowadays. For example, if you try
requests.get('http://edition.cnn.com'), you’re going to find that the response
contains very little usable data! It has become increasingly necessary to be able to exe‐
cute JavaScript locally in order to obtain data, because many sites use JavaScript to
load their actual content. The process of executing such JavaScript to produce the
final, complete HTML output is called rendering.

To accomplish rendering, we use a neat project called Splash, which describes itself as
a “JavaScript rendering service.” It can run in a Docker container and provides an API
for rendering other sites. Internally, it uses a (JavaScript-capable) WebKit engine to
fully load and render a website. This is what we’ll use to obtain website data. Our
aiohttp server, will call this Splash API to obtain the page
data.

## Codes Explanation

The news() function is the handler for the /news URL on our server. It returns the HTML page showing all the headlines.

Here, we have only two news websites to be scraped: CNN and Al Jazeera. More could easily be added, but then additional postprocessors would also have to be added, just like the cnn_articles() and aljazeera_articles() functions that are customized to extract headline data.

For each news site, we create a task to fetch and process the HTML page data for its front page. Note that we unpack the tuple ((*s)) since the news_fetch() coroutine function takes both the URL and the postprocessing function as parameters. Each news_fetch() call will return a list of tuples as headline results, in the form `<article URL>`, `<article title>`.

All the tasks are gathered together into a single Future (gather() returns a future representing the state of all the tasks being gathered), and then we immediately await the completion of that future. This line will suspend until the future completes.

Since all the news_fetch() tasks are now complete, we collect all of the results into a dictionary. Note how nested comprehensions are used to iterate over tasks,
and then over the list of tuples returned by each task. We also use f-strings to substitute data directly, including even the kind of page, which will be used in CSS to color the div background.

In this dictionary, the key is the headline title, and the value is an HTML string for a div that will be displayed in our result page.

Our web server is going to return HTML. We’re loading HTML data from a local file called index.html.

We substitute the collected headline div into the template and return the page to the browser client. This generates the page.

Here, inside the news_fetch() coroutine function, we have a tiny template for hitting the Splash API (which, for me, is running in a local Docker container on
port 8050). This demonstrates how aiohttp can be used as an HTTP client.

The standard way is to create a ClientSession() instance, and then use the get() method on the session instance to perform the REST call. In the next line,
the response data is obtained. Note that because we’re always operating on coroutines, with async with and await, this coroutine will never block: we’ll be able to
handle many thousands of these requests, even though this operation (news_fetch()) might be relatively slow since we’re doing web calls internally.

After the data is obtained, we call the postprocessing function. For CNN, it’ll be cnn_articles(), and for Al Jazeera it’ll be aljazeera_articles().

We have space only for a brief look at the postprocessing. After getting the page data, we use the Beautiful Soup 4 library for extracting headlines.

The match() function will return all matching tags (I’ve manually checked the HTML source of these news websites to figure out which combination of filters extracts the best tags), and then we return a list of tuples matching the format `<article URL>`, `<article title>`.

This is the analogous postprocessor for Al Jazeera. The match() condition is slightly different, but it is otherwise the same as the CNN one.
