In this exercise, we will work with a CSV file as our data source to create individual pages for each of the 45 United States presidents. The CSV file is provided for you: presidents.csv (Open it above.)

1. DOWNLOAD THE ZIPPED VERSION OF THIS REPO to follow along in class.

2. UNZIP it and drag the folder flask-exercise-master into the folder that contains your virtualenv for Flask projects.

3. ACTIVATE your virtualenv.

4. Our Flask app file is: presidents.py Open it in Atom.

These instructions are on a slide to show in the classroom.

A completed CSS file and 45 images have been provided in the static folder, which is where all such files (and JS files as well) must be for a Flask app to use them.

Three mostly completed Flask template files are in the templates folder, also required for a Flask app that uses templates.

NOTE: It is assumed you are in a Python 3.x virtual environment which has been activated and in which Flask has been installed.

Contents
Convert a CSV file to a dictionary
Test the dictionary list in a Flask route
Create a directory page (a list of links)
Examine the first template
Get the data needed for the directory page
Revisit the first template
Create a detail page
The detail page template
Relationships between routes, and templates, and links
Conclusion
Convert a CSV file to a dictionary
Our first task is to convert the CSV to a Python dictionary (or more accurately, a list of dictionaries).

A function to do this has already been written. It is in the modules.py file. It requires the built-in Python module csv. This script will work with any CSV file that has a header row.

import csv

def convert_to_dict(filename):
    """
    Convert a CSV file to a list of Python dictionaries.
    """
    # open a CSV file - note - must have column headings in top row
    datafile = open(filename, newline='')

    # create list of OrderedDicts as of Python 3.6
    my_reader = csv.DictReader(datafile)

    # write it all out to a new list
    list_of_dicts = []
    for row in my_reader:
        # we convert each row to a string and add a newline
        list_of_dicts.append( dict(row) )

    # close original csv file
    datafile.close()
    # return the list
    return list_of_dicts
Instead of copying this function into our Flask app file, presidents.py, we will import it there.

ACTION 1: Add this on line 2 in presidents.py:

from modules import convert_to_dict
Note that now you can run convert_to_dict() by entering any CSV filename as the argument. The function returns a list of dictionaries. With presidents.csv as the argument, the function returns a list of 45 dictionaries.

ACTION 2: Add this on line 5 in presidents.py ABOVE the route:

presidents_list = convert_to_dict("presidents.csv")
It is convenient to have presidents_list as a global variable so that we can use it in all of our Flask routes.

This Python list, presidents_list, contains 45 dictionaries — one per president.

Test the dictionary list in a Flask route
The Flask app (presidents.py) already has one simple route:

@app.route('/')
def index():
    return '<h1>Welcome to the presidential Flask example!</h1>'
ACTION 3: In presidents.py, change the route function index() to:

def index():
    heading = '<h1>Welcome to the presidential Flask example!</h1>'
    test1 = '<p>' + presidents_list[0]['President']
    test2 = ", born in " + presidents_list[0]['Birthplace'] + '.</p>'
    return heading + test1 + test2
We know that listname[0] will return the value of the first item in a Python list. In our list of dictionaries, presidents_list, each item is a complete dictionary of information about one U.S. president.

To access any item inside a dictionary, we use its key. Our keys in presidents_list include 'President' and 'Birthplace' (these key names came from the column headings in the CSV). We cannot access the dictionary with presidents_list['President'] because — remember — presidents_list is a LIST. So we access one item in the list and then the key inside that item: presidents_list[0]['President'].

ACTION 4: Save the edited presidents.py file and run it in Terminal (first, make sure your virtualenv is activated):

python presidents.py
That launches Flask's built-in local web server. In your web browser, type localhost:5000/ in the address bar to launch the web server — you will see the result of @app.route('/') and its function, index().

If your browser displayed "Welcome to the presidential Flask example!" and "George Washington, born in Westmoreland County, Virginia." — you have verified that you can access presidents_list from a route function. Review the function above to ensure you understand how it worked, because we're about to change it further.

You can view the CSV file (presidents.csv) as a lovely table here on GitHub and see all the presidential facts that are available to us from presidents_list, our list of dictionaries.

Create a directory page (a list of links)
In our app, there will be two page types:

First, a directory page or index, listing all presidents by name, in the order of their presidency. Each name will be a link that opens a president's detail page.
Second, the detail page. This will have the same layout and information for each individual president.
We will change the existing Flask route (index()) to create the directory page.

ACTION 5: Our first change is to add a template to the index() function. We already import render_template at the top of our app script, so all that's needed is to change the return statement, which currently reads:

return heading + test1 + test2
To this:

return render_template('index.html', pairs=pairs_list, the_title="Presidents Index")
How it works: Now, instead of writing the variables heading, test1 and test2 directly into the browser window, Flask will get a template file named index.html and write its contents into the browser window. The render_template function here passes two variables to the template: pairs and the_title.

We have not created pairs_list in presidents.py yet, so we can't run this yet. It would throw an error ("NameError: name 'pairs_list' is not defined").

Examine the first template
All Flask templates must be in the templates directory. Open the template named index.html and note the following within the HTML:

The H1 element
A P element
A UL element
One LI element inside the UL
The H1 and P elements have normal text in them. We can write anything in a template that we would write in any regular HTML file.

The UL element contains Jinja2 templating directives:

{% for pair in pairs %}
...
{% endfor %}
Those two directives are the start and end of a Python for-loop. If this reminds you of PHP (written inside HTML) — yes, it's the same idea. Flask allows us to insert Jinja2 directives to run Python commands in a template file. (For other Jinja2 directives, read the docs.)

We will loop over a list named pairs. Where is that list, and how did the template get access to it? We passed it to this template with return render_template(), covered above. We haven't yet written the code that creates pairs_list, but when we look at this for-loop, we can see what the list must contain:

{% for pair in pairs %}
    <li><a href="/president/{{ pair[0] }}">{{ pair[1] }}</a></li>
{% endfor %}
The double curly braces will be filled differently each time the loop repeats.

What we're aiming for is a list of 45 presidents in which each line looks like this:

<li><a href="/president/1">George Washington</a></li>
Each pair in the list needs to provide, first, the number of the presidency, and second, the full name of the president.

Let's return to the app file and write that into the route function.

Get the data needed for the directory page
ACTION 6: In presidents.py, in the route function, delete old code so that you're left with only this:

@app.route('/')
def index():
    # presidents_list[0]['President']
    # presidents_list[0]['Birthplace']

    return render_template('index.html', pairs=pairs_list, the_title="Presidents Index")
Do not delete anything above or below the route function!

We know we need two items (a pair!) of information for each president: the number of the presidency, and the full name of the president. Earlier, we got the name of the first president with presidents_list[0]['President']. We got his birthplace with presidents_list[0]['Birthplace']. We don't need his birthplace now, but we need to know which key to use to get the number of his presidency.
