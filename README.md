TEAM MEMBERS
-------------------------
* Susannah Liu @susannl5
   * NetID: susannl5
   * Student ID: 48294849
* Danny Cowles @cowlesd
   * NetID: cowlesd
   * Student ID: 13015455
* Zachary Qin @zacharyq
   * NetID: zacharyq
   * Student ID: 19291733
* Jalen Fu @jalenf
   * NetID: jalenf
   * Student ID: 44898079
  
ABOUT
-------------------------
This is the base implementation of a full crawler that uses a spacetime
cache server to receive requests.

Extra credit:

We check the html body of the robots.txt at every domain by downloading the text and then we use RobotFileParser and cache the results so that when accessing a url, we can check the domain and see if we are allowed to crawl that url.

We have implemented both a simhash to check for near duplicates and a checksum algorithm to check for similarities. If the current page is similar to a previously crawled page, we choose not to crawl it or include its stats in our report.

We also implemented multithreading with the politeness set to 500 ms with 4 threads and only up to two threads can access the same domain every 500 ms. To implement the politeness we have a shared dictionary keeping track of which thread is accessing which domain which is locked whenever accessed to prevent race conditions and make sure it is async-safe and thread-safe and if a third thread is attempting to access the domain, we set it to sleep for 500 ms.


Crawler Requirements:

The base code provided to us honors the politness delay for each site. Our implementation of our is_valid function allows us to crawl all pages with high textual information content along with detecting and avoiding infinite traps. Our crawler does not get stuck in any infinite traps with our implementation. One of the methods we use to avoid sets of similar pages with no information is by avoid pages past a certain set number. One of the issues we saw is that our crawler would retrieve pages from a certain site that went up to 200-300 pages long. These pages were very similar and didn't provide us much useful information so we avoid. In the extract_next_links function we examine the response code from the webpage download and if it is 204 (No Content) or anything larger than 400 we will not crawl it. This ensures that we are crawling webpages successfully based on status codes. In terms of avoiding large files we avoid specific file extensions that are not webpages in our is_valid function to ensure our crawler runs efficiently. One example of a large file we ran into were Powerpoint presentations so we had to avoid the extensions related to them, also since they are not webpages.


CONFIGURATION
-------------------------

### Step 1: Install dependencies

If you do not have Python 3.6+:

Windows: https://www.python.org/downloads/windows/

Linux: https://docs.python-guide.org/starting/install3/linux/

MAC: https://docs.python-guide.org/starting/install3/osx/

Check if pip is installed by opening up a terminal/command prompt and typing
the commands `python3 -m pip`. This should show the help menu for all the 
commands possible with pip. If it does not, then get pip by following the
instructions at https://pip.pypa.io/en/stable/installing/

To install the dependencies for this project run the following two commands
after ensuring pip is installed for the version of python you are using.
Admin privileges might be required to execute the commands. Also make sure
that the terminal is at the root folder of this project.
```
python -m pip install packages/spacetime-2.1.1-py3-none-any.whl
python -m pip install -r packages/requirements.txt
```

### Step 2: Configuring config.ini

Set the options in the config.ini file. The following
configurations exist.

**USERAGENT**: Set the useragent to `IR F19 uci-id1,uci-id2,uci-id3`. 
It is important to set the useragent appropriately to get the credit for 
hitting our cache.

**HOST**: This is the host name of our caching server. Please set it as per spec.

**PORT**: This is the port number of our caching server. Please set it as per spec.

**SEEDURL**: The starting url that a crawler first starts downloading.

**POLITENESS**: The time delay each thread has to wait for after each download.

**SAVE**: The file that is used to save crawler progress. If you want to restart the
crawler from the seed url, you can simply delete this file.

**THREADCOUNT**: This can be a configuration used to increase the number of concurrent
threads used. Do not change it if you have not implemented multi threading in
the crawler. The crawler, as it is, is deliberately not thread safe.


### Step 3: Define your scraper rules.

Develop the definition of the function scraper in scraper.py

```
def scraper (url: str, resp: utils.response.Response): -> list
    pass
```

The scraper takes in two parameters:

**ARGS**

*url*:

The URL that was added to the frontier, and downloaded from the cache.
It is of type str and was an url that was previously added to the
frontier.

*resp*:

This is the response given by the caching server for the requested URL.
The response is an object of type Response (see utils/response.py)
```
class Response:
    Attributes:
        url:
            The URL identifying the response.
        status:
            An integer that identifies the status of the response. This
            follows the same status codes of http.
            (REF: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
            In addition there are status codes provided by the caching
            server (600-606) that define caching specific errors.
        error:
            If the status codes are between 600 and 606, the reason for
            the error is provided in this attribute. Note that for status codes
            (400-599), the error message is not put in this error attribute; instead it
            must picked up from the raw_response (if any, and if useful).
        raw_response:
            If the status is between 200-599 (standard http), the raw
            response object is the one defined by the requests library.
            Useful resources in understanding this raw response object:
                https://realpython.com/python-requests/#the-response
                https://requests.kennethreitz.org/en/master/api/#requests.Response
            HINT: raw_response.content gives you the webpage html content.
```
**Return Value**

This function needs to return a list of urls that are scraped from the
response. (An empty list for responses that are empty). These urls will be
added to the Frontier and retrieved from the cache. These urls have to be
filtered so that urls that do not have to be downloaded are not added to the
frontier.

The first step of filtering the urls can be by using the **is_valid** function
provided in the same scraper.py file. Additional rules should be added to the is_valid function to filter the urls.

EXECUTION
-------------------------

To execute the crawler run the launch.py command.
```python3 launch.py```

You can restart the crawler from the seed url
(all current progress will be deleted) using the command
```python3 launch.py --restart```

You can specify a different config file to use by using the command with the option
```python3 launch.py --config_file path/to/config```

ARCHITECTURE
-------------------------

### FLOW

The crawler receives a cache host and port from the spacetime servers
and instantiates the config.

It launches a crawler (defined in crawler/\_\_init\_\_.py L5) which creates a 
Frontier and Worker(s) using the optional parameters frontier_factory, and
worker_factory.

When the crawler in started, workers are created that pick up an
undownloaded link from the frontier, download it from our cache server, and
pass the response to your scraper function. The links that are received by
the scraper is added to the list of undownloaded links in the frontier and
the url that was downloaded is marked as complete. The cycle continues until
there are no more urls to be downloaded in the frontier.

### REDEFINING THE FRONTIER:

You can make your own frontier to use with the crawler if they meet this
interface definition:
```
class Frontier:
    def __init__(self, config, restart):
        #Initializer.
        # config -> Config object (defined in utils/config.py L1)
        #           Note that the cache server is already defined at this
        #           point.
        # restart -> A bool that is True if the crawler has to restart
        #           from the seed url and delete any current progress.

    def get_tbd_url(self):
        # Get one url that has to be downloaded.
        # Can return None to signify the end of crawling.

    def add_url(self, url):
        # Adds one url to the frontier to be downloaded later.
        # Checks can be made to prevent downloading duplicates.
    
    def mark_url_complete(self, url):
        # mark a url as completed so that on restart, this url is not
        # downloaded again.
```
A sample reference is given in utils/frontier.py L10. Note that this
reference is not thread safe.

### REDEFINING THE WORKER

You can make your own worker to use with the crawler if they meet this
interface definition:
```
from scraper import scraper
from utils.download import download
class Worker(Thread): # Worker must inherit from Thread or Process.
    def __init__(self, worker_id, config, frontier):
        # worker_id -> a unique id for the worker to self identify.
        # config -> Config object (defined in utils/config.py L1)
        #           Note that the cache server is already defined at this
        #           point.
        # frontier -> Frontier object created by the Crawler. Base reference
        #           is shown in utils/frontier.py L10 but can be overloaded
        #           as detailed above.
        self.config = config
        super().__init__(daemon=True)

    def run(self):
        In loop:
            > url = get one undownloaded link from frontier.
            > resp = download(url, self.config)
            > next_links = scraper(url, resp)
            > add next_links to frontier
            > sleep for self.config.time_delay
```
A sample reference is given in utils/worker.py L9.

THINGS TO KEEP IN MIND
-------------------------

1. It is important to filter out urls that do not point to a webpage. For
   example, PDFs, PPTs, css, js, etc. The is_valid filters a large number of
   such extensions, but there may be more.
2. It is important to filter out urls that are not with ics.uci.edu domain.
3. It is important to maintain the politeness to the cache server (on a per
   domain basis).
4. It is important to set the user agent in the config.ini correctly to get
   credit for hitting the cache servers.
5. Launching multiple instances of the crawler will download the same urls in
   both. Mechanisms can be used to avoid that, however the politeness limits
   still apply and will be checked.
6. Do not attempt to download the links directly from ics servers.
