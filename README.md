# IntelX

IntelX is a Python command-line utility and API wrapper for intelx.io, made to perform any kind of open-source intelligence.

![](cli/screenshot1.png)

## Installation
```bash
git clone https://github.com/IntelligenceX/SDK
pip3 install SDK/Python
```

## Setup

To specify the API key to use, you can choose one of two options:
* Setting the ```INTELX_KEY``` environment variable.
* Manually supplying the ```-apikey``` argument.

You can get your API key here: https://intelx.io/account?tab=developer

##### Environment Variable
```bash
# create an INTELX_KEY env var with your API key.
export INTELX_KEY=00000000-0000-0000-0000-000000000000
```

##### Via the client

```bash
intelx.py -search riseup.net -apikey 00000000-0000-0000-0000-000000000000
```

## Configuration

On windows, we need to manually configure the command prompt/terminal in order to enable color support. You can do that with the following instructions:

1. Create following file ```Enable Color.reg```
```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Console]
"VirtualTerminalLevel"=dword:00000001
```

2. Right Click ```Enable Color.reg``` -> Merge

## Usage

```bash
intelx.py -search riseup.net
```


#### Quick search
```bash
intelx.py -search riseup.net
```

#### Quick search in buckets
```bash
intelx.py -search riseup.net -buckets "pastes, darknet.tor"
```

#### Search with 100 results
```bash
intelx.py -search riseup.net -limit 100
```

#### Download Item

The ```-download``` argument will set the HTTP request type to a stream, ultimately returning the raw bytes.
This allows us to download documents such as PDFs, ZIP, Word documents, Excel, etc.
You may set the filename with the ```-name``` argument.
```bash
# save item as test.pdf
intelx.py -download 29a97791-1138-40b3-8cf1-de1764e9d09c -name test.txt
```

#### View Item

To view the full data of a specific search result, specify the item's ID and use the `--view` parameter:

```bash
intelx.py -search 3a4d5699-737c-4d22-8dbd-c5391ce805df --view
```

#### Search Phonebook
```bash
intelx.py -search cia.gov --phonebook
```

#### Extract email from phonebook search
```bash
intelx.py -search cia.gov --phonebook --emails
```


## Usage as a library

**IntelX** is a double-edged sword. You can use it as a library for your own tooling, or you can use the command-line utility.

To use it as a library, all you have to do is import it in your project, and initialize the class. If you supply an API key, it will use that, if not, it will automatically select the public API key (limited functionality).

```python3
from intelx import intelx
intelx = intelx()
```

Once you’ve done that, you can use any of the functions defined in the class.

### Quick search

To execute a quick search, we can easily just use the intelx.search() function.

```python3
from intelx import intelx

intelx = intelx('abcd-efgh-ijkl-mnop')
search = intelx.search('hackerone.com')
```

### Advanced search

By default, the “maxresults” limit is set to 100 to avoid unnecessarily overloading the system. This value can be overridden at any time by setting the maxresults argument.

```python3
from intelx import intelx

intelx = intelx('abcd-efgh-ijkl-mnop')
search = intelx.search('hackerone.com', maxresults=200)
```

The following arguments have default values, but can be overridden to your choosing:


* maxresults=100
* buckets=[]
* timeout=5
* datefrom=””
* dateto=””
* sort=4
* media=0
* terminate=[]

#### Searching buckets

Lets say we want to search a term within specific buckets (leaks & darknet), we can modify our code to look something like this:

```python3
from intelx import intelx

b = ['darknet', 'leaks']

intelx = intelx('abcd-efgh-ijkl-mnop')
search = intelx.search('hackerone.com', maxresults=200, buckets=b)
```

#### Filtering by date

Another thing we can do is filter results based on their date.

When setting the dateto and datefrom options, it’s very important to note that you cannot use one without the other.
If you choose to use one, you must provide the other if you want it to work properly.

You must also accurately set the times as well.

```python3
from intelx import intelx

startdate = "2014-01-01 00:00:00"
enddate = "2014-02-02 23:59:59"

intelx = intelx('abcd-efgh-ijkl-mnop')

search = intelx.search(
   'riseup.net',
   maxresults=200,
   datefrom=startdate,
   dateto=enddate
)
```

#### Filtering by data type

We can filter results based on their data type using the “media” argument.

Using the following script, we can filter paste documents dated between 2014-01-01 and 2014-02-02 that have been collected.

You can find a table below with all the media types and their respective IDs.

```python3
from intelx import intelx

media_type = 1 # Paste document
startdate = "2014-01-01 00:00:00"
enddate = "2014-02-02 23:59:59"

intelx = intelx('abcd-efgh-ijkl-mnop')

search = intelx.search(
   'riseup.net',
   maxresults=200,
   media=media_type,
   datefrom=startdate,
   dateto=enddate
)
```

#### Search statistics

If we want to collect some statistics, we can use the following code:

```python3
from intelx import intelx

intelx = intelx('abcd-efgh-ijkl-mnop')

search = intelx.search(
   'riseup.net',
   maxresults=1000,
)

stats = intelx.stats(search)
print(stats)
```

### Viewing/reading files

There is one fundamental difference between the `FILE_VIEW` function and `FILE_READ` function. Viewing is for quickly viewing contents of a file (generally assumed to be text).
`FILE_READ`, on the other hand, is for direct data download.

This means if the resource is a ZIP/Binary or any other type of file, you can reliably get the contents without any encoding issues.

#### Viewing

```python3
from intelx import intelx

intelx = intelx()
search = intelx.search('riseup.net')

# grab file contents of first search result
contents = intelx.FILE_VIEW(search['records'][0]['storageid'])

print(contents)
```

It is worth noting that the view function accepts a “format” argument, which allows us to view the file in a different format.

For example, if we have a binary file and want to get a hex dump of it, we can set the format argument to “1”.

See Format Types for more.

```python3
from intelx import intelx

intelx = intelx()
search = intelx.search('riseup.net')

# grab file contents of first search result
contents = intelx.FILE_VIEW(search['records'][0]['storageid'], format=1)

print(contents)
```

For other format types, please refer to Media Types

#### Reading

If we want to download/read a file’s raw bytes, we can use the `FILE_READ` function.
The file will be saved as “file.contents”. If a name hasn’t been set, it will save as the storage id.

```python3
from intelx import intelx

intelx = intelx()
search = intelx.search('riseup.net')

# save the first search result file as "file.contents"
intelx.FILE_READ(search['records'][0]['storageid'], name="file.contents")
```

### Other Notes

#### Media Types

Here is a table listing the media types, along with their respective IDs.

| ID            | Media Type                         |
| ------------- | -----------------------------------|
| 0             | All                                |
| 1             | Paste document                     |
| 2             | Paste user                         |
| 3             | Forum                              |
| 4             | Forum board                        |
| 5             | Forum thread                       |
| 6             | Forum post                         |
| 7             | Forum user                         |
| 8             | Screenshot of website              |
| 9             | HTML copy of website               |
| 13            | Tweet                              |
| 14            | URL                                |
| 15            | PDF document                       |
| 16            | Word document                      |
| 17            | Excel document                     |      
| 18            | Powerpoint document                |    
| 19            | Picture                            |
| 20            | Audio file                         |
| 21            | Video file                         |
| 22            | Container file (ZIP/RAR/TAR, etc)  |
| 23            | HTML file                          |
| 24            | Text file                          |


#### Format Types

| ID |      Format Type                    |
|----|-------------------------------------|
| 0  |	textview of content                |
| 1  |	hex view of content                |
| 2  |	auto detect hex view or text view  |
| 3  |	picture view                       |
| 4  |	not supported                      |
| 5  | 	html inline view (sanitized)       |
| 6  |	text view of pdf                   |
| 7  |	text view of html                  |
| 8  |	text view of word file             |


## Contribute

* Issue Tracker: github.com/IntelligenecX/SDK
* Source Code: github.com/IntelligenecX/SDK

## Support

If you are having issues, please let us know.

## License

The project is licensed under the BSD license.

```

