## How the Internet Works

## Physical Layer

Most cities to each other via underwater cables: <https://www.submarinecablemap.com/>

These cables are laid by giant ships, with giant spools of cable:

<img width=400px src=img/ship1.jpg />

Notice the person for scale:

<img width=400px src=img/ship2.jpg />

Remotely operated vehicles (ROVs) walk along the ocean shore to monitor these cables:

<img width=400px src=img/rov.jpg />

Sometimes they break due to "acts of god":

<img width=400px src=img/shark.jpg />

Sometimes they break due to "acts of man":

<img width=400px src=img/missile.jpg />

Website to monitor internet outages: <https://netblocks.org/>

### Espionage

The US famously does not attack internet infrastructure to destroy it.
Instead it uses the internet to spy.

#### Operation Ivy Bells

Military communications also happen over undersea cables.

<img width=400px src=img/okhtosk.jpg />

<img width=400px src=img/artist-rendition.jpg />

<img width=400px src=img/inner-workings.jpg />

The operation was carried out by the USS Halibut:

<img width=400px src=img/uss-halibut-ssgn-587.jpg />

In 1981, Ronald Pelton sold details about Operation Ivy Bells to the Soviet embassy in Washington for $5000.

He was arrested in 1985 when KGB defector Vitaly Yurchenko informed the FBI.

<img width=400px src=img/pelton.jpg />

The wiretap device is now sitting in a museum in Moscow.

<img width=400px src=img/wiretap.jpg />

Find more details on wikipedia: <https://en.wikipedia.org/wiki/Operation_Ivy_Bells>

Operational costs estimated to be > $1 billion.

#### Modern NSA Wiretapping

[Room 641A](https://en.wikipedia.org/wiki/Room_641A) is a famous room in AT&T headquarters in San Francisco.

1. It contains wiretapping equipment for all internet traffic in/out of San Francisco.
2. The room was revealed by AT&T employee [Mark Klein](https://en.wikipedia.org/wiki/Mark_Klein) in 2006.

<img width=400px src=img/Room_641A_exterior.jpg />

Snowden leaks (2013)

1. FAIRVIEW/BLARNEY/STORBREW/OAKSTAR: Physically tap world-wide cables

    In general called "upstream collection"

    <img width=400px src=img/prism.png />

1. PRISM: NSA collects information from US companies using [FISA court orders](https://en.wikipedia.org/wiki/FISA_Amendments_Act_of_2008); started in 2007

    <img width=400px src=img/Prism-week-in-life-straight.png />

    <img width=400px src=img/Prism_slide_5.jpg />

1. MUSCULAR: NSA secretly wiretaps internal US company infrastructure

    <img width=400px src=img/NSA_Muscular_Google_Cloud.jpg />

## How Tracking Works

Reference: <https://www.wallarm.com/what/http-headers>

Every web connection has "header" information.

<img width=400px src=img/connection.jpg />

<img width=400px src=img/headers.jpeg />

Big list of user agents: <https://deviceatlas.com/blog/list-of-user-agent-strings>

### Anti-scraping Trick 1: headers

Websites can use headers to block scrapers.

```
>>> import requests
>>> requests.utils.default_headers()
{'User-Agent': 'python-requests/2.32.5', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive'}
```

You can change the user agent to work around this blocking.
([example reference](https://www.zenrows.com/blog/python-requests-user-agent#set-random-user-agents))
```
import requests
import random

# create a User Agent list
user_agent_list = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:139.0) Gecko/20100101 Firefox/139.0",
    "Mozilla/5.0 (iPhone; CPU iPhone OS 18_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.5 Mobile/15E148 Safari/604.1",
]

random.shuffle(user_agent_list)
user_agent = user_agent_list[0]

# use the randomized User Agent
headers = {"User-Agent": user_agent}
response = requests.get(url, headers=headers)
```

### Anti-scraping Trick 2: IP blocking

Some webpages will block your IP address if you connect to many times.

The only solution to this is to get a new IP address:

1. All campus wireless shares an IP, but wired connections on campus all have unique IPs.  So plug your laptop into a physical ethernet port, or use a lab machine.

2. Go to a starbucks out in town and use their wireless.

3. Use a VPN.

### Anti-scraping Trick 2b: rate limiting

Most websites allow you to connect at most:

1. 10x per second
1. 100x per minute
1. 1000x per day

If you exceed any of these limits, then 

### Anti-scraping Trick 3: Javascript

Some webpages serve Javascript instead of HTML.

A *headless web browser* can convert the javascript into HTML.

The most popular python library for headless browsers in python is called `playwright`.
You can download it with
```
$ pip3 install playwright
```
Then you need to register firefox with playwright using the command
```
$ playwright install firefox
```
The following function is like `requests.get`, but uses a headless webbrowser.
It is much slower because running javascript is slow.
```
from playwright.sync_api import sync_playwright
def download_html_and_run_javascript(url):
    with sync_playwright() as p:
        browser = p.firefox.launch(headless=True)
        page = browser.new_page()
        page.goto(url)
        page.wait_for_load_state("networkidle")
        html = page.content()
        browser.close()
    return html
```
Here is example usage for the ebay scraper assignment.
```
>>> url = 'https://www.ebay.com/sch/i.html?_nkw=hammer&_sacat=0&_from=R40&_trksid=m570.l1313&LH_TitleDesc=0&_odkw=hammer'
>>> html = download_html_and_run_javascript(url)
>>> len(html)
2594309
>>> from bs4 import BeautifulSoup
>>> soup = BeautifulSoup(html, 'html.parser')
>>> tags = soup.select('.s-card__title')
>>> len(tags)
94
>>> tags[30].text
'4LB LEAD HAMMER 2PCS FOR WIRE WHEELS KNOCK OFFSOpens in a new window or tab'
```

### Anti-scraping Trick 4: Fingerprinting

Javascript leaks a lot of information besides just headers that can be used to uniquely identify users.

See <https://amiunique.org/fingerprint>.

These "fingerprints" can be used to guess if a connection is from a human or a bot.

The [undetected-playwright](https://github.com/QIN2DIM/undetected-playwright?tab=readme-ov-file#sync) python library can be used to provide javascript fingerprints that "look human".

For the ebay assignment, you may need to modify `download_html_and_run_javascript` to use the undetected-playwright library.
