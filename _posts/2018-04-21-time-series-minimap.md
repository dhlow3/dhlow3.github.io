---
title: "A Time Series Mini-map with matplotlib"
---

A mini-map is a miniature map placed at the corner of a screen. Its purpose is
to allow the user to see where they are relative to some broader context. Mini-maps
are popular in real-time, strategy video games and can often be found in
modern text editors and IDEs.

In this post, we'll build the following mini-map visualization:

![Image of final mini-map viz]({{ site.baseurl }}/assets/images/posts/time-series-minimap/Lines%2C%20Columns%2C%20and%20Insets_13_0.png)
<!--sep-->

# Setup

## Imports

Along with a standard Python installation, we'll also need the pandas and
matplotlib libraries.

This is what our import statement looks like:
```python
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import matplotlib.patches as mpatches
import pandas as pd
from datetime import timedelta
from math import floor, log10
from matplotlib.ticker import FuncFormatter
```

## Number Formatter

I'm going to use a function to format my numbers. I like to shorten large
numbers in my visualizations to help with readability. The following function will
replace zeros and commas from large numbers with the appropriate number unit.

```python
def _millify(n, pos=None):
    """Big number formatter."""
    units = ['', ' K', ' M', ' B', ' T']
    n = float(n)
    i = max(0, min(len(units)-1, floor(0 if n == 0 else log10(abs(n))/3)))
    if n > 1000000:
        return '{:.1f}{}'.format(n / 10**(3 * i), units[i])
    else:
        return '{:.0f}{}'.format(n / 10**(3 * i), units[i])
```

## Data

I'll be using fabricated website traffic data, but any
time series with period-over-period data should work.

Let's pull in the data and look at the first five rows.
```python
data_file = './traffic_data.csv'
df = pd.read_csv(data_file, encoding='utf-8-sig')
df['Date'] = df['Date'].apply(pd.to_datetime)
df.set_index('Date', inplace=True)
df.head()
```


|    | Selected | 52 Weeks Prior
---- | -------- | -------------
Date | |
2016-01-01 | 44281 | 52336
2016-01-02 | 51781 | 51402
2016-01-03 | 59068 | 67024
2016-01-04 | 53968 | 63194
2016-01-05 | 52871 | 63641

# Build the Visualization

## Last 90 Days Plot

To begin building the visualization, we'll plot the last 90 days from
the dataset.

Let's find the last 90 days within our Date index.

```python
today = df.index.max()
lookback = timedelta(days=90)
start_of_lookback = today - lookback
```

Then, we define our line's horizontal positioning across the x axis with the
Date index. The vertical positioning is defined along the y axis with the values
in the Selected data column.

```python
subset = df[df.index > start_of_lookback]
main_x = subset.index
line = subset['Selected']
```

Now, we create a figure and plot our line.

```python
# Set up figure
fig = plt.figure()
ax = fig.add_subplot(111)
fig.set_figwidth(fig.get_figheight() * 3)
plt.ylabel('Traffic')

# Plot line
ax.plot(main_x, line, color='green', alpha=0.6, label='Selected')

# Axes formatting
ax.set_xlim(xmin=start_of_lookback, xmax=today)
ax.xaxis.set_major_locator(mdates.WeekdayLocator(byweekday=1, interval=2))
ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %d'))
formatter = FuncFormatter(_millify)
ax.yaxis.set_major_formatter(formatter)

# Remove top and right spines and ticks
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')

plt.show()
```

![Image of 90 day line plot]({{ site.baseurl }}/assets/images/posts/time-series-minimap/Lines%2C%20Columns%2C%20and%20Insets_11_0.png)

## Line and Column Plot

Next, we'll plot the entire dataset using columns to show the last 52 weeks (2016)
and a line to show the 52 weeks prior (2015).

```python
# Define x and y positions
x = df.index
bar_height = df['Selected']
line1 = df['52 Weeks Prior']
```

```python
# Set up figure
fig = plt.figure()
ax1 = fig.add_subplot(111)
fig.set_figwidth(fig.get_figheight() * 3)
fig.subplots_adjust(bottom=0.2)
plt.ylabel('Traffic')

# Plot columns and line
ax1.bar(x, bar_height, color='green', alpha=0.6, width=0.6, label='Selected')
ax1.plot(x, line1, color='purple', label='52 Weeks Prior')

# Axes 1 formatting
ax1.set_xlim(xmin=df.index.min(), xmax=df.index.max())
ax1.xaxis.set_major_locator(mdates.WeekdayLocator(byweekday=1, interval=2))
ax1.xaxis.set_major_formatter(mdates.DateFormatter('%d'))
formatter = FuncFormatter(_millify)
ax1.yaxis.set_major_formatter(formatter)

# Remove top and right spines and ticks
ax1.spines['right'].set_visible(False)
ax1.spines['top'].set_visible(False)
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')

# Add a legend across the top
ax1.legend(
    bbox_to_anchor=(0.5, 1.25), loc=9, ncol=2, frameon=False, fontsize=10
    )

# 2nd axes to show month-year under the first x axis
ax2 = ax1.twiny()
ax2.xaxis.set_ticks_position("bottom")
ax2.xaxis.set_label_position("bottom")
ax2.spines["bottom"].set_position(("axes", -0.1))
ax2.set_xlim(ax1.get_xlim())
ax2.set_xticklabels(df.index)
ax2.xaxis.set_major_locator(mdates.MonthLocator(interval=2))
ax2.xaxis.set_major_formatter(mdates.DateFormatter('%b %Y'))

# Remove all second axes spines
for _ in ax2.spines:
    ax2.spines[_].set_visible(False)

plt.show()
```


![Image of line and column plot]({{ site.baseurl }}/assets/images/posts/time-series-minimap/Lines%2C%20Columns%2C%20and%20Insets_9_0.png)

### Shading

We'll want to highlight the 90 day period that is being focused on. To do this,
we use the axvspan method and set the start and end points along the x axis.

This time, we'll also build the legend using handles and labels. We add this step
so we can call out the highlighted region within the legend.

We just need to replace this section of code from the previous plot:

```python
# Add a legend across the top
ax1.legend(
    bbox_to_anchor=(0.5, 1.25), loc=9, ncol=2, frameon=False, fontsize=10
    )
```

With this section of code:

```python
# Shade the last 90 days in red
ax1.axvspan(start_of_lookback, today, alpha=0.25, color='red')

# Build a legend
handles, labels = ax1.get_legend_handles_labels()
red_patch = mpatches.Patch(color='red', alpha=0.25, label='Prior 90 Days')
handles = handles + [red_patch]
ax1.legend(bbox_to_anchor=(0.5, 1.25),
           loc=9,
           ncol=3,
           frameon=False,
           fontsize=10,
           handles=handles)
```


![Image of plot with red shading]({{ site.baseurl }}/assets/images/posts/time-series-minimap/Lines%2C%20Columns%2C%20and%20Insets_12_0.png)

## Final Mini-map Plot

Lastly, we create another axes for the mini-map and then put everything together
for the final plot.

```python
# Set up figure
fig = plt.figure()
ax = fig.add_subplot(111)
fig.set_figwidth(fig.get_figheight() * 3)
plt.ylabel('Traffic')

# Plot line
ax.plot(main_x, line, color='green', alpha=0.6, label='Selected')

# Axes formatting
ax.set_xlim(xmin=start_of_lookback, xmax=today)
ax.xaxis.set_major_locator(mdates.WeekdayLocator(byweekday=1, interval=2))
ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %d %Y'))
formatter = FuncFormatter(_millify)
ax.yaxis.set_major_formatter(formatter)

# Remove top and right spines and ticks
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.xaxis.set_ticks_position('bottom')
ax.yaxis.set_ticks_position('left')

# Add another axes for the inset plot
ax1 = fig.add_axes([0.8, 0.75, 0.45, 0.3])

# Plot columns and line
ax1.bar(x, bar_height, color='green', alpha=0.5, width=1, label='Selected')
ax1.plot(x, line1, color='purple', label='52 Weeks Prior')

# Axes 1 formatting
ax1.set_xlim(xmin=df.index.min(), xmax=df.index.max())
ax1.xaxis.set_major_locator(mdates.WeekdayLocator(byweekday=1, interval=3))
ax1.xaxis.set_major_formatter(mdates.DateFormatter('%d'))
formatter = FuncFormatter(_millify)
ax1.yaxis.set_major_formatter(formatter)

# Remove top and right spines and ticks
ax1.spines['right'].set_visible(False)
ax1.spines['top'].set_visible(False)
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')

# Shade the last 90 days in red
ax1.axvspan(start_of_lookback, today, alpha=0.25, color='red')

# Build a legend
handles, labels = ax1.get_legend_handles_labels()
red_patch = mpatches.Patch(color='red', alpha=0.25, label='Prior 90 Days')
handles = handles + [red_patch]
ax1.legend(bbox_to_anchor=(0.5, 1.25),
           loc=9,
           ncol=3,
           frameon=False,
           fontsize=10,
           handles=handles)

# 2nd axes to show month-year under the first x axis
ax2 = ax1.twiny()
ax2.xaxis.set_ticks_position("bottom")
ax2.xaxis.set_label_position("bottom")
ax2.spines["bottom"].set_position(("axes", -0.1))
ax2.set_xlim(ax1.get_xlim())
ax2.set_xticklabels(df.index)
ax2.xaxis.set_major_locator(mdates.MonthLocator(interval=2))
ax2.xaxis.set_major_formatter(mdates.DateFormatter('%b %Y'))

# Remove all second axes spines
for _ in ax2.spines:
    ax2.spines[_].set_visible(False)

plt.show()
```

![Image of final mini-map viz]({{ site.baseurl }}/assets/images/posts/time-series-minimap/Lines%2C%20Columns%2C%20and%20Insets_13_0.png)

# Conclusion
In this post, we built a visualization that imitates a mini-map you might find
in a real-time, strategy video game or modern IDE. Again, the idea here is to
allow the user to understand the greater context of whatever data they are
looking at. The next steps for this visualization would be to build
functionality that would let the user move around the plot to look at time
periods outside of just the most recent 90 days. Maybe I'll tackle that in
another post.
