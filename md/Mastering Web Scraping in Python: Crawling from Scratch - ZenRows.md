> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zenrows.com](https://www.zenrows.com/blog/mastering-web-scraping-in-python-crawling-from-scratch)

> Learn how to build a website crawler for scraping at scale. Start by visiting pages one by one and sc......

Have you ever tried to crawl thousands of pages? Scale that even further? Handle and recover from system failures?

After seeing how to [extract content from a website](https://www.zenrows.com/blog/mastering-web-scraping-in-python-from-zero-to-hero) and [how to avoid being blocked](https://www.zenrows.com/blog/stealth-web-scraping-in-python-avoid-blocking-like-a-ninja), we'll take a look at the crawling process. To get data at scale, getting a few URLs by hand is not an option. We need to use an automated system that will discover new pages and visit them.

_Disclaimer: for real-world usage, find suitable software. Below is more info on that. This guide pretends to be an introduction to how the crawling process works and doing the basics. But there are tons of details that need addressing._

### Prerequisites

For the code to work, you will need [python3 installed](https://www.python.org/downloads/). Some systems have it pre-installed. After that, install all the necessary libraries by running `pip install`.

```
pip install requests beautifulsoup4
```

How to Get all the Links on the Page
------------------------------------

From the first article in the series, we know that getting data from a webpage is easy with `requests.get` and `BeautifulSoup`. We will start by finding the links in a [fake shop prepared for testing scraping](https://scrapeme.live/shop/page/1/).

The basics to get the content are the same. Then we get all the links on the paginator and add the links to a `set`. We chose set to avoid duplicates. As you can see, we hardcoded the selector for the links, meaning that it is not a universal solution. For the moment, we'll focus on the page at hand.

```
import requests 
from bs4 import BeautifulSoup 
 
to_visit = set() 
response = requests.get('https://scrapeme.live/shop/page/1/') 
soup = BeautifulSoup(response.content, 'html.parser') 
for a in soup.select('a.page-numbers'): 
	to_visit.add(a.get('href')) 
 
print(to_visit)
```

One URL at a Time, Sequential
-----------------------------

Now we have several links but no way to visit them all. We need some kind of loop that will execute the extracting part for every URL available to fix that. Maybe the most straightforward way, although not the scalable one, is to use the same loop. But before that, there is a missing piece: avoid crawling the same page twice.

We'll keep track of already visited links in another `set` and avoid duplicates by checking them before every request. In this case, `to_visit` is not being used, just maintained for demo purposes. To prevent visiting every page, we'll also add a `max_visits` variable. For now, we ignore the `robots.txt` file, but we have to be civil and nice.

```
visited = set() 
to_visit = set() 
max_visits = 3 
 
def crawl(url): 
	print('Crawl: ', url) 
	response = requests.get(url) 
	soup = BeautifulSoup(response.content, 'html.parser') 
	visited.add(url) 
	for a in soup.select('a.page-numbers'): 
		link = a.get('href') 
		to_visit.add(link) 
		if link not in visited and len(visited) < max_visits: 
			crawl(link) 
 
crawl('https://scrapeme.live/shop/page/1/') 
 
print(visited) 
print(to_visit)
```

It is a recursive function with two exit conditions: there are no more links to visit, or we reached the maximum visits. In either case, it will exit and print the visited links and the ones pending.

It is important to note that the same link can be added many times, but it will only get crawled once. In a big project, the idea would be to set a timer and only request each URL after a few days.

Separation of Concerns
----------------------

We said this is not about extracting or parsing content, but we need to separate concerns before it becomes entangled. For that, we'll create three helper functions: get HTML, extract links, and extract content. As their names imply, each of them will perform one of the main tasks of web scraping.

The first one will get the HTML from a URL using the same library as earlier but wrapping it in a `try` block for security.

```
def get_html(url): 
	try: 
		return requests.get(url).content 
	except Exception as e: 
		print(e) 
		return ''
```

The second one, extracting the links, will work just as before.

```
def extract_links(soup): 
	return [a.get('href') for a in soup.select('a.page-numbers') 
		if a.get('href') not in visited]
```

The last one will be the placeholder for extracting the content we want. Since we are simplifying this part, it will get basic info from the same page, no need to enter on the detail page.  
To show that we can extract some content, we will print each product's title (Pokémon name).

```
def extract_content(soup): 
	for product in soup.select('.product'): 
		print(product.find('h2').text)
```

Assembling it all together.

```
def crawl(url): 
	if not url or url in visited: 
		return 
	print('Crawl: ', url) 
	visited.add(url) 
	html = get_html(url) 
	soup = BeautifulSoup(html, 'html.parser') 
	extract_content(soup) 
	links = extract_links(soup) 
	to_visit.update(links)
```

Noticed something different? The crawling logic is not attached to the link extracting part. Each of the helpers handles a single piece. And the `crawl` function acts as an orchestrator by calling them and applying the results.

As the project evolves, all these parts could be moved to files or passed as parameters/callbacks. We can generalize the use cases if the core is independent of the selected page and content.

Are we missing something? 🤔  
We need to add the first URL and call the crawling function. Since `crawl` is not recursive anymore, we'll handle that in a separate loop.

```
to_visit.add('https://scrapeme.live/shop/page/1/') 
 
while (len(to_visit) > 0 and len(visited) < max_visits): 
	crawl(to_visit.pop())
```

Parallel Requests
-----------------

There is a significant part missing: parallelism. HTTP request handlers are idle most of the time, waiting for the response to come back. It means that we can send several of them at the same time without overloading the machine. And then process them as they came back.

It is relevant to note that this approach only works if the order is not imperative. But we are already using sets, which according to [Python's definition](https://docs.python.org/3/tutorial/datastructures.html#sets), "a set is an **unordered collection** with no duplicate elements." Meaning that our process was unordered from the start.

Before diving deep into the parallel requests, we have to understand a couple of concepts: synchronization and queues.

### Synchronized Queues

There is a huge risk in threaded or [parallel computing](https://en.wikipedia.org/wiki/Parallel_computing): modifying the same variables or data structures from different threads. It means two of our requests would be adding new links to a set (i.e., `to_visit`). Since the data structure is not protected, both could read and write it like this:

*   Both read its content, i.e. `(1, 2, 3)` _(simplified)_
*   Thread one adds links to pages `4, 5`: `(1, 2, 3, 4, 5)`
*   Thread two adds links to pages `6, 7`: `(1, 2, 3, 6, 7)`

How did this happen? When thread two wrote the new links, it added them to a set with only three elements.  
_This is a very simplified version; check the links for more info._

What can we do to avoid these conflicts? Synchronization or locking. From the [docs](https://docs.python.org/3/library/queue.html): "queues use locks to temporarily block competing threads." It means that thread one would acquire a lock on the set, read and write without any problem, and then release the lock automatically. Meanwhile, thread two would have to wait until the lock becomes available. Only then read and write.

```
import queue 
 
q = queue.Queue() 
q.put('https://scrapeme.live/shop/page/1/') 
 
def crawl(url): 
	... 
	links = extract_links(soup) 
	for link in links: 
		if link not in visited: 
			q.put(link)
```

For the moment, it does not work. Do not worry. The changes in the existing code are minimum: we replaced `to_visit` with a queue. But queues need handlers or workers to process their content. With the above, we have created a Queue and added an item (the original one). We also modified the `crawl` function to put links in the queue instead of updating the previous set.

We'll create a worker using the [threading module](https://docs.python.org/3/library/threading.html) to process that queue.

```
from threading import Thread 
 
def queue_worker(i, q): 
	while True: 
		url = q.get() 
		print('to process:', url) 
		q.task_done() 
 
q = queue.Queue() 
Thread(target=queue_worker, args=(0, q), daemon=True).start() 
 
q.put('https://scrapeme.live/shop/page/1/') 
q.join() 
print('Done')
```

We defined a new function that will handle the queued items. For that, we enter into an infinite loop that will stop when all the processing finishes.

Then `get` an item, which will block until an item is available. We process that item; for the moment, just print it to show how it works. It will call `crawl` later.

Finally, we notify the queue that the item has been processed by calling `task_done`.

Once the queue gets notified for all the items and empty, it will stop its execution and end the infinite loop. That's what the `join` function does, "blocks until all items in the queue have been gotten and processed."

Now we need two more things: process items and create more threads (it would not be parallel with just one, would it?).

```
def queue_worker(i, q): 
	while True: 
		url = q.get() 
		if (len(visited) < max_visits and url not in visited): 
			crawl(url) 
		q.task_done() 
 
q = queue.Queue() 
num_workers = 4 
for i in range(num_workers): 
	Thread(target=queue_worker, args=(i, q), daemon=True).start()
```

Be careful when running it since big numbers in `num_workers` and `max_visits` would start lots of requests. If the script had some minor bug for any reason, you could perform hundreds of requests in a few seconds.

### Performance

We run benchmarks with different settings only as a reference.

*   Sequential requests: 29,32s
*   Queue with one worker (`num_workers = 1`): 29,41s
*   Queue with two workers (`num_workers = 2`): 20,05s
*   Queue with five workers (`num_workers = 5`): 11,97s
*   Queue with ten workers (`num_workers = 10`): 12,02s

There is almost no difference between sequential requests and having one worker. Threads carry some overhead, but it is barely noticeable here. It would require a more severe load test. Once we start adding workers, that overhead pays off. We could add even more, but it won't affect the outcome since they will be idle most of the time.

Distributed Processing
----------------------

We won't cover the following scale-up step: distributing the crawling process among several servers. [Python allows it](https://docs.python.org/3/library/multiprocessing.html#using-a-remote-manager), and some libraries can help you with it ([Celery](https://docs.celeryproject.org/en/stable/) or [Redis Queue](https://python-rq.org/)). It is a huge step, and we have already covered enough for the day.

As a quick preview, the idea behind it is the same as the one with the threads. Each item will be processed as we've seen until now but in different threads or even machines running the same code. With this approach, we can scale even further; theoretically, with no limit. But in reality, there is always a limit or bottleneck, usually the central node that handles the distribution.

Take into Account when Scaling Up
---------------------------------

We've shown a simplified version of a crawling process for educational purposes. To apply all this at scale, you should consider several things first.

### Build vs Buy vs Open Source

Before you write your own library for crawling, try some of the options out there. Many great Open Source libraries can achieve it: [Scrapy](https://docs.scrapy.org/en/latest/), [pyspider](http://docs.pyspider.org/en/latest/), [node-crawler](https://github.com/bda-research/node-crawler) (Node.js), or [Colly](https://github.com/gocolly/colly) (Go). And many companies and services that provide you with scraping and crawling solutions.

### Avoid being blocked

As we saw in a previous post, [there are several actions we can take to avoid blocking](https://www.zenrows.com/blog/stealth-web-scraping-in-python-avoid-blocking-like-a-ninja). A couple of them are proxies and headers. Here is a simple snippet adding those to our current code.  
_Note that these [free proxies](https://free-proxy-list.net/) might not work for you. They are short-time lived._

```
proxies = { 
	'http': 'http://190.64.18.177:80', 
	'https': 'http://49.12.2.178:3128', 
} 
 
headers = { 
	'authority': 'httpbin.org', 
	'cache-control': 'max-age=0', 
	'sec-ch-ua': '"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"', 
	'sec-ch-ua-mobile': '?0', 
	'upgrade-insecure-requests': '1', 
	'user-agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36', 
	'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9', 
	'sec-fetch-site': 'none', 
	'sec-fetch-mode': 'navigate', 
	'sec-fetch-user': '?1', 
	'sec-fetch-dest': 'document', 
	'accept-language': 'en-US,en;q=0.9', 
} 
 
def get_html(url): 
	try: 
		response = requests.get(url, headers=headers, proxies=proxies) 
		return response.content 
	except Exception as e: 
		print(e) 
		return ''
```

We won't go into details here, only a simple snippet for extracting id, name, and price per item. We store everything in a `data` array, which is not a great idea. But it is enough for demo purposes.

```
data = [] 
 
def extract_content(soup): 
	for product in soup.select('.product'): 
		data.append({ 
			'id': product.find('a', attrs={'data-product_id': True})['data-product_id'], 
			'name': product.find('h2').text, 
			'price': product.find(class_='amount').text 
		}) 
 
print(data)
```

### Persistency

We haven't persisted anything, and that does not scale. In a real-world case, we should store the content and even the HTML itself for later processing. And all the discovered URLs with the timestamp time. It all starts to sound like a database is needed. Depending on the necessities, we could store just the actual content or the whole URLs, dates, HTML, etcetera generically.

### Canonicals

The link extraction part does not take into consideration [canonical links](https://en.wikipedia.org/wiki/Canonical_link_element). A page can have more than one URL: query strings or hashes might modify it. In our case, we would crawl it twice. It's not a problem now, but something to consider.

The right approach would be to add the canonical URL (if present) to the visited list. Then we could arrive at that same page from a different origin URL, but we would detect it as duplicate. We could also remove some query string parameters using [url_query_cleaner](https://w3lib.readthedocs.io/en/latest/w3lib.html#w3lib.url.url_query_cleaner).

### Robots.txt

We have not checked it because we are using a test website prepared for scraping. But please check the robots file and comply with it when crawling an actual target. And above it, do not cause more traffic than they can handle. Once again, be civil and nice ;)

Final Code
----------

```
import requests 
from bs4 import BeautifulSoup 
import queue 
from threading import Thread 
 
starting_url = 'https://scrapeme.live/shop/page/1/' 
visited = set() 
max_visits = 100 
num_workers = 5 
data = [] 
 
def get_html(url): 
	try: 
		response = requests.get(url) 
		
		return response.content 
	except Exception as e: 
		print(e) 
		return '' 
 
def extract_links(soup): 
	return [a.get('href') for a in soup.select('a.page-numbers') 
			if a.get('href') not in visited] 
 
def extract_content(soup): 
	for product in soup.select('.product'): 
		data.append({ 
			'id': product.find('a', attrs={'data-product_id': True})['data-product_id'], 
			'name': product.find('h2').text, 
			'price': product.find(class_='amount').text 
		}) 
 
def crawl(url): 
	visited.add(url) 
	print('Crawl: ', url) 
	html = get_html(url) 
	soup = BeautifulSoup(html, 'html.parser') 
	extract_content(soup) 
	links = extract_links(soup) 
	for link in links: 
		if link not in visited: 
			q.put(link) 
 
def queue_worker(i, q): 
	while True: 
		url = q.get() 
		if (len(visited) < max_visits and url not in visited): 
			crawl(url) 
		q.task_done() 
 
q = queue.Queue() 
for i in range(num_workers): 
	Thread(target=queue_worker, args=(i, q), daemon=True).start() 
 
q.put(starting_url) 
q.join() 
 
print('Done') 
print('Visited:', visited) 
print('Data:', data)
```

Conclusion
----------

We'd like you to part with three main points:

1.  Separate getting the HTML and extracting the links from the crawling itself.
2.  Choose the appropriate system for your use case: simple sequential, parallel, or distributed.
3.  Building from scratch to a vast scale will probably hurt. Take a look at free or paid libraries/solutions.

Do not forget to take a look at the previous posts in this series.  
+ [Stealth Scraping in Python: Avoid Blocking Like a Ninja](https://www.zenrows.com/blog/stealth-web-scraping-in-python-avoid-blocking-like-a-ninja) (Part II)  
+ [Mastering Extraction](https://www.zenrows.com/blog/mastering-web-scraping-in-python-from-zero-to-hero) (Part I)  

Stay tuned for the next one on scaling this crawling process even further.

Did you find the content helpful? Spread the word and share it on [Twitter](https://twitter.com/share?text=%E2%9C%8D%F0%9F%8F%BBMastering%20Web%20Scraping%20in%20Python:%20Crawling%F0%9F%95%B8%EF%B8%8F.%20Learn%20how%20to%20build%20a%20website%20crawler%20for%20scraping%20at%20scale%F0%9F%94%A5&url=https%3A%2F%2Fwww.zenrows.com%2Fblog%2Fmastering-web-scraping-in-python-crawling-from-scratch%2F%3Futm_source%3Dtwitter%26utm_medium%3Dshared%26rd%3D1878717786), [LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fwww.zenrows.com%2Fblog%2Fmastering-web-scraping-in-python-crawling-from-scratch%2F%3Futm_source%3Dlinkedin%26utm_medium%3Dshared%26rd%3D264523195) or [Facebook](https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.zenrows.com%2Fblog%2Fmastering-web-scraping-in-python-crawling-from-scratch%2F%3Futm_source%3Dfacebook%26utm_medium%3Dshared%26rd%3D457353703).

**Never Get Blocked Again.** Our API handles rotating proxies and headless browsers for you.

[Try Now for free](https://app.zenrows.com/register)