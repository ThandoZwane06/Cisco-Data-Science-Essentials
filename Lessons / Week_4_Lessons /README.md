# Paris Airbnb Listings: Data Visualization

A data visualization mini project using the Maven Analytics Airbnb listings dataset, filtered down to Paris. Built as practice applying chart storytelling techniques (reference lines, conditional bar coloring, spine removal, annotations) covered this week in Cisco Networking Academy's Data Science Essentials with Python course.

## Dataset

Public multi-city Airbnb listings and reviews dataset from Maven Analytics, filtered to a single city (Paris) for this project.

## Data cleaning and preparation

Before any charting, the raw data needed cleaning and reshaping:

- Checked for missing values across the dataset with `.isna().sum()` and `.describe()` to get a first sense of data quality
- Queried for logically invalid rows, e.g. listings where `accommodates == 0` or `price == 0`, and rows where `host_since` was `NaN`, to understand what needed handling before analysis
- Cast date columns to proper datetime format with `pd.to_datetime()`, applied to both the listings (`host_since`) and reviews (`date`) tables
- Checked which cities were present in the dataset with `.value_counts()` on the `city` column, then filtered down to Paris only using `.query("city == 'Paris'")`
- Selected only the columns needed for this analysis (`host_since`, `neighbourhood`, `city`, `accommodates`, `price`) to keep the working dataframe focused

## Feature engineering

- Extracted the year a host joined from `host_since` using `.dt.year`, cast to `Int64` to handle the type cleanly
- Grouped by that joined year to count new hosts per year (`.groupby('year_joined')['neighbourhood'].count()`) and calculate the average listing price per year (`.groupby('year_joined')['price'].mean().round(2)`)
- Combined both into a single summary dataframe (`paris_list_overtime`) with `new_hosts` and `avg_price` columns, used for the two time-series charts below
- Grouped by neighbourhood to get average price per neighbourhood (`.groupby('neighbourhood')['price'].mean().round(2)`), sorted and reset the index for charting
- Filtered to the most expensive neighbourhood (Elysee) and grouped by `accommodates` to get average price by accommodation capacity within that neighbourhood

## Charts

### 1. Average Airbnb Price Over Time

```python
plt.plot(paris_list_overtime.index, paris_list_overtime['avg_price'])
plt.axvspan(2009, 2014, color='red', alpha=0.05)
plt.xlabel('Year')
plt.ylabel('Average Price (Euros)')
plt.title('Average AirBnB Price Over Time')
plt.gca().spines[['top', 'right']].set_visible(False)
```

Average nightly price spikes sharply around 2009 (close to €160), declines steadily through 2014 (down to roughly €100), then climbs back up through 2018-2020, before a sharp drop in 2021. Used `axvspan` to shade the 2009-2014 decline period, drawing the eye to that stretch specifically rather than leaving the viewer to spot it unaided. Also removed the top and right spines to keep the chart cleaner and less boxed-in.

### 2. New Airbnb Hosts Over Time

```python
plt.plot(paris_list_overtime.index, paris_list_overtime['new_hosts'])
#highlight the sharp decrease
plt.axvspan(2015, 2017, color='red', alpha=0.05)
plt.xlabel('Year')
plt.ylabel('New Hosts')
plt.title('New AirBnB Hosts Over Time')
plt.gca().spines[['top', 'right']].set_visible(False)
```

New host growth climbs rapidly from 2008, peaking around 2015 (over 12,000 new hosts that year), then falls sharply. Shaded the 2015-2017 window specifically to highlight that sharp decline, same reasoning as the price chart, letting the shading do some of the storytelling instead of relying purely on the line shape.

### 3. Average Price by Accommodation Capacity

```python
#convert accommodates column to a string
paris_list_accom['accommodates'] = paris_list_accom['accommodates'].astype(str)

plt.figure(figsize=(6,4))
plt.barh(paris_list_accom['accommodates'], paris_list_accom['price'])
plt.xlabel('Price per Night (Euros)')
plt.ylabel('Accommodation Capacity')
plt.title('Average Price by Accommodation Capacity')
```

Converted `accommodates` to a string before plotting, since it needed to be treated as a discrete category on the axis rather than a continuous number. Price generally rises with capacity, but not perfectly linearly, a 14-person listing had the highest average price per night, even ahead of 16-person listings, worth digging into further as a possible outlier or small sample size within that group.

### 4. Average Price by Neighbourhood

```python
#mean line
mean_price = paris_list_neigh['price'].mean().round()

#set the bar colors depending on the condition
bar_colors = ['C0' if price < mean_price else 'C4' for price in paris_list_neigh['price']]

plt.figure(figsize=(10,6))
plt.barh(paris_list_neigh['neighbourhood'], paris_list_neigh['price'], color=bar_colors)

#add the vertical line
plt.axvline(x=mean_price, color='grey', linestyle='--', label='Average Price')

#add the text
plt.text(x=mean_price+3, y=0, s=f'{mean_price}', color='black', fontweight='bold')

#remove the spines
plt.gca().spines[['top','right']].set_visible(False)

#add labels
plt.xlabel('Price per Night (Euros)')
plt.ylabel('Neighbourhood')
plt.title('Average Price by Neighbourhood')
plt.legend()
plt.show()
```

The most detailed chart of the four. Bars are conditionally colored, above-average neighbourhoods in one color, below-average in another, using a list comprehension against the calculated mean price. Added a dashed vertical reference line (`axvline`) at the average price so each bar can be judged against that benchmark rather than in isolation, plus a text label calling out the exact average value (€124). Elysee came out highest (around €210/night), Menilmontant lowest (around €75/night).

## What I learned this week

- **Chart storytelling techniques**: using `axvspan` to shade a specific time range and draw attention to it, `axvline` to add a reference line for comparison, and `text()` to place a specific value directly on the chart rather than making the viewer read it off the axis.
- **Conditional bar coloring**: building a list of colors with a list comprehension based on a condition (`'C0' if price < mean_price else 'C4'`), then passing that list directly into `color=` for a `barh()` call.
- **Removing chart spines** (`plt.gca().spines[['top','right']].set_visible(False)`) to reduce visual clutter and make a chart feel less boxed-in, a small but consistent presentation choice used across all four charts.
- **`UnicodeDecodeError` when loading data**: `pd.read_csv()` defaults to UTF-8 encoding, and international text (accented characters like é, ç) can break that assumption and crash the read. Fixed by explicitly passing the correct `encoding` parameter.
- Reinforced `.query()` for filtering rows with readable conditional strings, and `.groupby()` combined with `.count()` / `.mean()` for summarizing data by category.

## Tools

Python, pandas, matplotlib
