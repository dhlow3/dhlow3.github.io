---
title: "Quick Create SQL Table - Part 2"
---

As a follow up to [Quick Create SQL Table - Part 1]({% post_url 2018-04-15-quick-create-sql-table-part-1 %}),
I'll walk through the graphical user interface (GUI) I built using the [appJar](http://appjar.info/) library.
With a GUI, we should be able to produce a CREATE SQL TABLE statement more quickly than with
the command line tool we used in Part 1. Additionally, a GUI should make this tool more user friendly as we won't need to type
out file paths or use the `--help` argument to see what optional arguments are available.

![Image of app]({{ site.baseurl }}/assets/images/posts/quick-create-part-2/app-unfilled.png)

<!--sep-->

## Requirements
### appJar
Aside from Python and a data file, the only other requirement to get the
GUI up and running is the appJar library.

appJar according to the [appJar](http://appjar.info/) website is...
>The easiest way to create GUIs in Python.
>Written by a teacher, in the classroom, for students.
>appJar is designed to run on as many versions of Python as possible - so it should work in your school.
>There are no other dependencies - simply download, unzip, and put it in your code folder.
Check out the installation instructions for other ways to get appJar working.

I'm a big fan of pip, so I used `pip install appjar` to get appJar into my
virtualenv.

### Dataset
Keeping with the theme from [Part 1]({% post_url 2018-04-15-quick-create-sql-table-part-1 %}),
we'll use another beer dataset from [Kaggle](https://www.kaggle.com/nickhould/craft-cans/data).
This time we're looking at craft, canned beers from the US. The following is a sample from the dataset:

| | abv | ibu | id | name | style | brewery_id | ounces
- | --- | --- | -- | ---- | ----- | ---------- | ------
0 | 0.05 |  | 1436 | Pub Beer | American Pale Lager | 408 | 12
1 | 0.066 |  | 2265 | Devil's Cup | American Pale Ale (APA) | 177 | 12
2 | 0.071 |  | 2264 | Rise of the Phoenix | American IPA | 177 | 12
3 | 0.09 |  | 2263 | Sinister | American Double / Imperial IPA | 177 | 12
4 | 0.075 |  | 2262 | Sex and Candy | American IPA | 177 | 12

Alright, so it looks like we have a row number column for column one and at least
some missing values in the ibu column. We'll definitely want to keep these things in mind
when we go to build a table.

## Building the GUI
### Setup
First, we need a couple of imports, including the `parse_file` function we built in
[Part 1]({% post_url 2018-04-15-quick-create-sql-table-part-1 %}).

```python
from appJar import gui
from create_sql import parse_file
from os import path
```

Next, create a gui object.

```python
app = gui('Create SQL Table')
```

Now for some entry fields and labels. The first argument for each of these is
where we specify a name that can be used for calling the label, or entry, whenever
we need to modify it in some way.

```python
app.addLabel('file_label', 'Select Data File to Process')
app.addFileEntry('data_file')

app.addLabel('n_label', 'Enter number of rows to parse')
app.addNumericEntry('n')
app.setEntryDefault('n', 'leave blank for all')

app.addLabel('sep_label', 'Choose how the data is separated')
app.addOptionBox('sep', ['tab', 'space', '|', ':', ',', ';'])
```

Next, create a check box. Here, the label is the first argument, and the
check box name is the second argument.

```python
app.addNamedCheckBox('Show commented data examples ', 'eg')
```

The following is a brief explanation for how the two entries and boxes will be used:
* `addFileEntry` allows us to search the file drive for our data file
* `addNumericEntry` allows us to enter the number of rows we want to parse
* `addOptionBox` lets us choose how the entries in our data file are separated
* `addNamedCheckbox` will be where we pick whether or not we want commented examples

Now, add `app.go()`, name the script, and run it from the terminal.
I named my script app.py.

Here's what it looks like when I run `python app.py`:

![Image of GUI entries]({{ site.baseurl }}/assets/images/posts/quick-create-part-2/entries.png)

With this, we can add our data file, and set our options in the GUI.

### Add Buttons
Let's add three buttons.

```python
app.addButtons(['Process', 'Reset', 'Exit'], [process, reset, exit])
```
The addButtons method takes a list of button labels for the first argument. The
second argument is a list of functions that should be called when the associated
buttons are clicked.

#### Exit and Reset Buttons
Let's define these functions by starting with what should happen when a user clicks the
exit button.

```python
def exit():
    """Define action for exit button."""
    app.stop()
```

For the reset button, we want to clear all of the entries and then reset the focus
back to the first entry.

```python
def reset():
    """Define action for reset button."""
    app.clearAllEntries()
    app.clearOptionBox('sep')
    app.setCheckBox('eg', ticked=False)
    app.setFocus('data_file')
```

#### Process Button
The process button is definitely the more complicated piece of the entire app.
This will be where we call the parse_file function from [Part 1]({% post_url 2018-04-15-quick-create-sql-table-part-1 %}).
We'll go over the process function in two steps.

##### Step 1 - Grab the Input
```python
def process():
    """Define action for process button."""
    parse_args = app.getAllEntries()
    parse_args['sep'] = app.getOptionBox('sep')
    parse_args['eg'] = app.getCheckBox('eg')

    # Convert tab and space for processing
    if parse_args['sep'] == 'tab':
        parse_args['sep'] = '\t'
    elif parse_args['sep'] == 'space':
        parse_args['sep'] = ' '

    # Validate data_file and n entries
    invalid = False
    if not path.exists(parse_args['data_file']):
        app.errorBox('file_error', 'Invalid file')
        invalid = True

    if parse_args['n']:
        try:
            parse_args['n'] = int(parse_args['n'])
        except ValueError:
            app.errorBox('n_error', 'Invalid number of rows to parse')
            invalid = True

        if parse_args['n'] < 0:
            app.errorBox('n_error', 'Invalid number of rows to parse')
            invalid = True
```

There are three main pieces to the first step of the process function as we've
just defined it.
1. grab the entry/box inputs and store in a dictionary
2. convert the words "tab" and "space" to \t and ' ' so `parse_file()` can interpret them
3. do some validation

##### Step 2 - Parse the File

```python
if not invalid:
    # If no error messages, output results to sub-window
    app.startSubWindow('Output', modal=True)

    try:
        # Do file parsing
        text = parse_file(parse_args)
    except Exception:
        app.stop()
        raise

    app.addLabel('result')
    app.setLabel('result', text)

    app.addButtons(['Back', 'Copy Text', 'Exit '],
                   [back, copy_text(text), exit])

    app.stopSubWindow()
    app.go(startWindow='Output')
```

If our validation from step 1 passes, we want to create a sub-window, parse the
file, and return the results to the sub-window.

Additionally, we're going to add three buttons to the sub-window, one of which
we've already defined (exit).

##### Copy Text Button

When we parse the file, the results are stored as a string in a variable named text.
We can pass the text variable as an argument to our copy text button. This will allow
us to copy the results to the clipboard so we can paste them elsewhere e.g.
query editor, text file, .sql file, etc.

```python
def copy_text(text):
    """Copy text to the clipboard."""
    app.topLevel.clipboard_append(text)
```

##### Back Button

The back button does three things.
1. clear the clipboard
2. destroy the sub-window
3. show the main window

```python
def back():
    """Exit output window and return to main app."""
    app.topLevel.clipboard_clear()
    app.destroySubWindow('Output')
    app.show()
```

## Usage

If we've done everything correctly, we should have a usable GUI for producing
`CREATE TABLE` SQL statements.

### Input
![Image of App Entries Filled]({{ site.baseurl }}/assets/images/posts/quick-create-part-2/app-filled.png)

### Output
![Image of Parse File Results]({{ site.baseurl }}/assets/images/posts/quick-create-part-2/results.png)

Now we can copy the text into a query window, change data types as necessary,
and name the table. We would probably want to do something with that nameless
first column as well. As a note, the ibu column is missing a commented example because
their was no value for ibu in the first row of the data file. This first row is where
examples are pulled from.

## Conclusion
In this post, we were able to reuse the parse_file function we had written in
[Part 1]({% post_url 2018-04-15-quick-create-sql-table-part-1 %}) and create a GUI
with [appJar](http://appjar.info/). This makes for a quicker, more user friendly method for building
SQL tables from data files.

To see the full code, visit the [create-sql-state GitHub Repository](https://github.com/dhlow3/create-sql-state).
