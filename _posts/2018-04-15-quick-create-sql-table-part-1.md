---
title: "Quick Create SQL Table - Part 1"
---

I often find myself needing to build a SQL table from a flat file.  This can
 happen for multiple reasons. Sometimes, I need to join the data in the file
 with a database table for the purpose of doing a quick ad hoc analysis. Other
 times, I need to build an ETL process from some external data source. Understanding
 what kind of data will be coming in can often be difficult if only a sample
 flat file is available, especially if it is a very large flat file.

There are various methods for importing a flat file into a SQL table, but at
 some point, you'll need to define your data types. Using VARCHAR(255) for
 everything is an option. Sure, you could do that, but what if
 there was a way to programmatically determine the column names, data types,
 and see example data? In this post, I'll walk through a Python command line
 tool I made to do just that. <!--sep-->

## Requirements
### Python
The command line tool is actually quite basic. There are four Python
 imports and two functions, all contained in one file, create_sql.py. The
 imports are included in a basic Python installation, so as long as you have
 Python installed, you can use this tool.

### Dataset
The next thing you'll need is a delimited, flat file containing a header row.
 For this post, I'll be using the comma-delimited, beer recipe dataset from
 [Kaggle](https://www.kaggle.com/jtrofe/beer-recipes/data). This dataset
 contains 22 columns and over 73 thousand rows.

To give you an idea of what we'll be working with, the following are the first
 couple of columns and rows from recipeData.csv:

BeerID | Name | URL | Style | StyleID | ...
------ | ---- | --- | ----- | ------- | ---
1 | Vanilla Cream Ale | /homebrew/recipe/view/1633/vanilla-cream-ale | Cream Ale | 45 | ...
2 | Southern Tier Pumking clone | /homebrew/recipe/view/16367/southern-tier-pumking-clone	Holiday/Winter | Special Spiced Beer | 85 | ...
3 | Zombie Dust Clone - EXTRACT | /homebrew/recipe/view/5920/zombie-dust-clone-extract | American IPA | 7 | ...
4 | Zombie Dust Clone - ALL GRAIN | /homebrew/recipe/view/5916/zombie-dust-clone-all-grain | American IPA | 7 | ...
5 | Bakke Brygg Belgisk Blonde 50 l | /homebrew/recipe/view/89534/bakke-brygg-belgisk-blonde-50-l | Belgian Blond Ale | 20 | ...

## Using argparse
The first function we'll use is `get_args()`. It utilizes the
 [argparse](https://docs.python.org/3/library/argparse.html) library to get
 arguments passed from the command line. It then returns the arguments in a
 dictionary.

Let's start by building an ArgumentParser object.
```python
parser = argparse.ArgumentParser(
    description='Build SQL create table statement.')
```
 Then, we'll define one required argument and three optional arguments.
```python
parser.add_argument('data_file',
                    help='a delimited data file')

parser.add_argument('-n',
                    type=int,
                    help='number of rows to parse (default: all rows)')

parser.add_argument('-sep',
                    dest='sep',
                    default='\t',
                    help='delimiter (default: \\t)')

parser.add_argument('-eg',
                    dest='eg',
                    action='store_true',
                    default=False,
                    help=('show example data as comments in SQL '
                          'statement'))
```
Next, we call the ```parse_args()``` method. According to the
 [docs](https://docs.python.org/3/library/argparse.html),
 >This will inspect the command line, convert each argument to the appropriate type and then invoke the appropriate action.

```python
parser_args = parser.parse_args()
```
Finally, we store all of the arguments in a dictionary, and just to be safe,
 make sure that the data file location actually exists.
```python
args = dict(data_file=parser_args.data_file,
            n=parser_args.n,
            sep=parser_args.sep,
            eg=parser_args.eg)

assert path.exists(args['data_file'])
```
Here is a breakdown of the arguments:
1. data_file - the location of our delimited data file and the only required argument
2. n - the number of lines to look at when parsing the file
 * the default is to parse the entire file
3. sep - this is where we define how the file is delimited
 * the default is tab-delimited
4. eg - see the first value from each row as a comment in our resulting create table statement
 * the default does not show examples

## File Parsing
Up to this point, we have a function that takes arguments from the command line
 and stores them in a dictionary. Now, we need to do something with the data
 file. The ```parse_file(args)``` function takes the args dictionary and returns
 a ```CREATE TABLE``` SQL statement.

First, we open the data file, create a ```DictReader()``` object, and read the
 header row from the file.
```python
with open(args['data_file']) as data_file:
    currentline = 0

    reader = DictReader(data_file, delimiter=args['sep'])
    header = reader.fieldnames
```

Next, create an ```OrderedDict()``` object, iterate through the DictReader i.e.
 our data_file, and grab the max text length in each column. We'll store these
 lengths in our OrderedDict. If we pass -eg in the command line,
 we'll also store the values from the first data row.
```python
attr = OrderedDict()

try:
    for row in reader:
        currentline += 1
        if args['n']:
            if currentline > args['n']:
                break

        for name in header:
            val = row[name]

            if currentline == 1:
                attr[name] = {'ex': val, 'len': len(val)}

            if len(val) > attr[name]['len']:
                attr[name] = {'ex': attr[name]['ex'], 'len': len(val)}

except UnicodeDecodeError:
    # Highlight location of UnicodeDecodeError in exception message
    print('UnicodeDecodeError AT LINE {} OF DATA FILE'.format(
        currentline))
    raise
```
Lastly, we take the attr OrderedDict and transform it to a nicely formatted
 string so we can use the results in a query.
```python
cols = ''
max_key_len = max(map(len, attr))  # for pretty spacing in SQL statement

for _ in attr.keys():
    if args['eg']:
        cols = (cols +
                '{}{}NVARCHAR({}),\t-- eg. {}\n'
                .format(_,
                        ' ' * (max_key_len - len(_) + 8),
                        attr[_]['len'],
                        attr[_]['ex']))

        sql = "CREATE TABLE []\n(\n{});\n".format(cols)
    else:
        cols = (cols +
                '{}{}NVARCHAR({}),\n'
                .format(_,
                        ' ' * (max_key_len - len(_) + 8),
                        attr[_]['len']))

        sql = "CREATE TABLE []\n(\n{});\n".format(cols)
```
## Usage
Okay, so that's nice and all, but what does this tool look like in action? First,
 navigate to the location where the create_sql.py file is located. Then, open
 the command prompt/terminal in this location. From here, if you type ```python
 create_sql.py --help```, you'll get a nice message reminding you what this tool
 is and what arguments can be passed.

![Image of help message]({{ site.baseurl }}/assets/images/posts/quick-create-part-1/help.png)

WARNING, this script does not take file encoding into consideration.
 As long as there are at least a couple of rows to parse before running into```UnicodeDecodeError```,
 we'll have some data to begin building our create table statement.

Lets do a test run of the recipeData.csv and see what happens.

![Image of error message]({{ site.baseurl }}/assets/images/posts/quick-create-part-1/error.png)

There appears to be a decode error at line 40. In that case, I'll just parse the
 first 39 lines.

![Image of error message]({{ site.baseurl }}/assets/images/posts/quick-create-part-1/results.png)

Hey, hey, not too bad, but let's see what the -eg argument is all about.

![Image of error message]({{ site.baseurl }}/assets/images/posts/quick-create-part-1/results_eg.png)

## Conclusion
In this post, we built a command line tool that will parse a data file and output
 a ```CREATE TABLE``` SQL statement that we can use as a starting point for
 building a SQL table. Since we were unable to parse the entire file in the
 example we looked at, there may indeed be incorrect VARCHAR lengths. Hopefully,
 in most cases, that will not be an issue. Using the -eg argument, we have
 insight into what the data actually looks like. At this point, I would copy
 the output from the console and paste it in a SQL query. Then, I'd probably
 change some of the data types that don't need to be VARCHAR. For instance, I'd
 change BeerID to INT. Finally, I would replace the [] in ```CREATE TABLE []```
 with an actual table name, and run the query. Now, we have a table into which
 we can import all of our flat file data.

To see the full code, visit the [create-sql-state GitHub Repository](https://github.com/dhlow3/create-sql-state).
