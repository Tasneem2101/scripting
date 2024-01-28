# Parsing with Scrapy and Xpath
Exercise for scraping using scrapy, parsing a real website, extracting key information that is not available through an API and using SQL to query it later. Again, you will use an already existing local HTML file that was previously downloaded and available in this repo. This prevents websites changing and breaking the parsing in this repository.

Objective: to extract the names and medal (or position) from the top three High Jumpers in the World Junior Championships as presented in their respective Wikipedia pages.

Install _requirements.txt_ in a virtual environment

```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```


Get started with scrapy: https://docs.scrapy.org/en/latest/intro/tutorial.html

## Use the scrapy shell

To help you building the scraping, use the scrapy shell (an interactive scraping shell) to work with the HTML files. The scrapy CLI allows you to use absolute paths. Go to the root of this repo and run the following to open up the local HTML file:

```
scrapy shell  "https://en.wikipedia.org/wiki/Sport"
```

Note that the quotes are required for the shell to work and escape the single quote within the HTML page. The shell uses a IPython as the shell (Jupyter-like output in the terminal) so be aware that when copying and pasting, you might need to reformat.

Confirm that the `response.url` points to the local path:

```
In [1]: response
Out[1]: <200 https://en.wikipedia.org/wiki/Sport>
```

Make it easy to read and type by assigning the interesting table to a variable.

```
In [2]: table=response.xpath('//table')
In [3]: table            
out[3]:
[<Selector query='//table' data='<table class="wikitable">\n<tbody><tr>...'>,
 <Selector query='//table' data='<table class="nowraplinks mw-collapsi...'>,
 <Selector query='//table' data='<table class="nowraplinks mw-collapsi...'>,
 <Selector query='//table' data='<table class="nowraplinks hlist mw-co...'>]
```

That shows one table that have the data we need. 
```
In [5]: rows = table.xpath('//tr')
In [6]:  row = rows[2]
In [8]:  row.xpath('td//text()')[0].extract()
Out[8]: 'Cricket'
```
That looks just right.

Now put everything together to loop over the tables and extract the information:

```
In [24]: names=[]
    ...: for count,row in enumerate(response.xpath('//table//tbody//tr')[1:]):
    ...:     name={
    ...:     'Rank':count+1,
    ...:     'Sport':row.xpath('td[1]//text()').extract_first(),
    ...:     'Estimated Global Following' : row.xpath('td[2]//text()').extract_first(),
    ...:     'Sphere of Influence':row.xpath('td[3]//text()').extract_first(),}
    ...:     print(name)
    ...:     names.append(name)
    ...:     if count ==9:
    ...:         break

{'Rank': 1, 'Sport': 'Association football', 'Estimated Global Following': '4 billion', 'Sphere of Influence': 'Globally\n'}
{'Rank': 2, 'Sport': 'Cricket', 'Estimated Global Following': '2.5 billion', 'Sphere of Influence': 'primarily '}
{'Rank': 3, 'Sport': 'Hockey', 'Estimated Global Following': '2 billion', 'Sphere of Influence': 'Europe'}
{'Rank': 4, 'Sport': 'Tennis', 'Estimated Global Following': '1 billion', 'Sphere of Influence': 'Globally\n'}
{'Rank': 5, 'Sport': 'Volleyball', 'Estimated Global Following': '900 million', 'Sphere of Influence': 'Americas, Europe, Asia, Oceania\n'}
{'Rank': 6, 'Sport': 'Table tennis', 'Estimated Global Following': '875 million', 'Sphere of Influence': 'Mainly East Asia\n'}
{'Rank': 7, 'Sport': 'Basketball', 'Estimated Global Following': '825 million', 'Sphere of Influence': 'Globally\n'}
{'Rank': 8, 'Sport': 'Baseball', 'Estimated Global Following': '500 million', 'Sphere of Influence': 'primarily '}
{'Rank': 9, 'Sport': 'Rugby', 'Estimated Global Following': '475 million', 'Sphere of Influence': 'primarily UK, Ireland, France, Italy, Oceania, South Africa, Argentina, and Japan.\n'}
{'Rank': 10, 'Sport': 'Golf', 'Estimated Global Following': '450 million', 'Sphere of Influence': 'primarily Western Europe, '}
```

Now Saving the selected data into Json File 

```
In [26]: import json

In [27]: json_data=json.dumps(names)

In [28]: json_data
Out[28]: '[{"Rank": 1, "Sport": "Association football", "Estimated Global Following": "4 billion", "Sphere of Influence": "Globally\\n"}, {"Rank": 2, "Sport": "Cricket", "Estimated Global Following": "2.5 billion", "Sphere of Influence": "primarily "}, {"Rank": 3, "Sport": "Hockey", "Estimated Global Following": "2 billion", "Sphere of Influence": "Europe"}, {"Rank": 4, "Sport": "Tennis", "Estimated Global Following": "1 billion", "Sphere of Influence": "Globally\\n"}, {"Rank": 5, "Sport": "Volleyball", "Estimated Global Following": "900 million", "Sphere of Influence": "Americas, Europe, Asia, Oceania\\n"}, {"Rank": 6, "Sport": "Table tennis", "Estimated Global Following": "875 million", "Sphere of Influence": "Mainly East Asia\\n"}, {"Rank": 7, "Sport": "Basketball", "Estimated Global Following": "825 million", "Sphere of Influence": "Globally\\n"}, {"Rank": 8, "Sport": "Baseball", "Estimated Global Following": "500 million", "Sphere of Influence": "primarily "}, {"Rank": 9, "Sport": "Rugby", "Estimated Global Following": "475 million", "Sphere of Influence": "primarily UK, Ireland, France, Italy, Oceania, South Africa, Argentina, and Japan.\\n"}, {"Rank": 10, "Sport": "Golf", "Estimated Global Following": "450 million", "Sphere of Influence": "primarily Western Europe, "}]'

In [29]: with open ('data.json','w') as f:
    ...:     json.dump(names,f)
    ...:

 ```

 So the `b` tag is showing "Bronze". Update the loop to use the value of the `b` tag instead of the `bgcolor` value:

 ```
 In [81]: for tr in table.xpath('tr'):
             print(tr.xpath('td/b/text()').extract()[0],
             tr.xpath('td/a/text()').extract()[0]
             )

Gold Steve Smith
Silver Tim Forsyth
Bronze Takahiro Kimino
```

