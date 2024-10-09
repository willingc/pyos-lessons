---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.4
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

+++ {"editable": true, "slideshow": {"slide_type": ""}}

# Activity 3: Tests & Checks for your code

* In [activity 1](../activity-1/clean-code-activity-1), you took some code and made it cleaner using expressive variable names and docstrings to document the module. 
* In [activity 2](../activity-2/clean-code-activity-2), you made your code more DRY ("Don't Repeat Yourself") using documented functions and conditionals. 

In this activity, you will build checks into your workflow to handle data processing "features". 


### Real world data processing & workflows and edge cases 
Real-world data rarely can be imported without "work arounds". You will often find unusual data entries and values you don't expect. Sometimes, these values are documented - for example, a 9999 may represent a missing value in a dataset. Other times, there are typos and other errors in the data that you need to handle. Sometimes, call these unusual values or instances in a dataset or workflow "edge cases".  

Writing robust code that handles unexpected values will make your code run smoothly and fail gracefully. This type of code, which combines functions (or classes) and checks within the functions that handle messy data, will make your code easier to maintain over time. 

:::{tip}
Using functions, classes, and methods (functions within a class) is a great first step in handling messy data. A function or method provides a modular unit you can test outside of the workflow for the edge cases you may encounter. Also, because a function is a modular unit, you can add elements to handle unexpected processing features as you build your workflow.
:::

something about debuggers? 
* https://jupyterlab.readthedocs.io/en/stable/user/debugger.html

## Manage the unexpected 

In this activity, you will apply the following strategies:

* [conditional statements](../checks-conditionals/python-conditionals)
* try/except blocks

to process the JOSS citation data. 

:::{todo}
What branch is the lesson with try/except // ask for forgiveness, checks elements in??
IN THIS PR:
https://github.com/pyOpenSci/lessons/pull/14/files#diff-7f4ff1b75e85d38f3955cca051e68e8746773c279b34c9a0a400b9c2dc1240ff 
:::

When you can, try to use the Pythonic approach of asking for forgiveness later (ie use try/except blocks) rather than conditional statements. 

```{code-cell} ipython3
---
editable: true
slideshow:
  slide_type: ''
---
# This works but is less pythonic
def clean_title(title):
    """Notice that this function checks explicitly to see if it's a list and then processes the data.
    """
    if isinstance(title, list):
        return title[0]
    return title
```

## More "pythonic" - ask for forgiveness 

easier to ask for forgiveness

```{code-cell} ipython3
---
editable: true
slideshow:
  slide_type: ''
---
# This is the preferred way to catch an error 
def clean_title(title):
    """
    It's more Pythonic to try first and then ask for forgiveness later. 
    If you are writing tests this also makes your code easier to test. 
    """
    try:
        return title[0]
    except (TypeError, IndexError):
        return title
```

+++ {"editable": true, "slideshow": {"slide_type": ""}, "tags": ["hide-output", "hide-cell"]}

TODO - move this to activity 2 it doesn't belong in activity 3! 

### Applying functions to dataframe values - .apply 

The `.apply()` function in pandas allows you to apply any function to rows or columns in a `pandas.DataFrame`. For example, You can use it to perform operations on specific column or row values. When you use .apply(), you can specify whether you want to apply the function across columns (axis=0, the default) or across rows (axis=1). For example, if you want to apply a function to each row of a DataFrame, you would use df.apply(your_function, axis=1). This function is especially useful for applying logic that can’t be easily achieved with built-in pandas functions, allowing for more flexibility in data processing.

You can use `.apply` in pandas as an efficient replacement for `for loops` to process row and column values in a df.

```{code-cell} ipython3
a=1
```

+++ {"editable": true, "slideshow": {"slide_type": ""}}

The code below is an example of what your code might look like after completing activity 2.

You can choose to work with this code, or you can use the code that you completed in activity 2.

### What's changed

For this activity, you now have a new data file to open in your list of .json files. This file has some unexpected "features" that you will need to ensure your code handles gracefully. 

Your goal is similar to ensure that the code below runs. 

:::{tip}
The code below will fail. You will likely want to use a debugger to determine why it's failing and get the code running. 
:::

+++ {"editable": true, "slideshow": {"slide_type": ""}, "tags": ["raises-exception"]}

```python
import json
from pathlib import Path

import pandas as pd


def load_clean_json(file_path, columns_to_keep):
    """
    Load JSON data from a file. Drop unnecessary columns and normalize
    to DataFrame.

    Parameters
    ----------
    file_path : Path
        Path to the JSON file.
    columns_to_keep : list
        List of columns to keep in the DataFrame.

    Returns
    -------
    dict
        Loaded JSON data.
    """

    with file_path.open("r") as json_file:
        json_data = json.load(json_file)
    normalized_data = pd.json_normalize(json_data)

    return normalized_data.filter(items=columns_to_keep)


def format_date(date_parts: list) -> str:
    """
    Format date parts into a string.

    Parameters
    ----------
    date_parts : list
        List containing year, month, and day.

    Returns
    -------
    str
        Formatted date string.
    """
    return f"{date_parts[0]}-{date_parts[1]:02d}-{date_parts[2]:02d}"


def clean_title(value):
    """A function that removes a value contained in a list."""
    return value[0]


def process_published_date(date_parts):
    """Parse a date provided as a list of values into a proper date format.

    Parameters
    ----------
    date_parts : str or int
        The elements of a date provided as a list from CrossRef

    Returns
    -------
    pd.datetime
        A date formatted as a pd.datetime object.
    """

    date_str = (
        f"{date_parts[0][0]}-{date_parts[0][1]:02d}-{date_parts[0][2]:02d}"
    )
    return pd.to_datetime(date_str, format="%Y-%m-%d")


columns_to_keep = [
    "publisher",
    "DOI",
    "type",
    "author",
    "is-referenced-by-count",
    "title",
    "published.date-parts",
]

data_dir = Path("data")

all_papers_list = []
for json_file in data_dir.glob("*.json"):
    papers_df = load_clean_json(json_file, columns_to_keep)

    papers_df["title"] = papers_df["title"].apply(clean_title)
    papers_df["published_date"] = papers_df["published.date-parts"].apply(
        process_published_date
    )

    all_papers_list.append(papers_df)

all_papers_df = pd.concat(all_papers_list, axis=0, ignore_index=True)

print("Final shape of combined DataFrame:", all_papers_df.shape)
```

```{code-cell} ipython3
---
editable: true
slideshow:
  slide_type: ''
---

```

```{code-cell} ipython3
---
editable: true
slideshow:
  slide_type: ''
---

```