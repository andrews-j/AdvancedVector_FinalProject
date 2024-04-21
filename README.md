# Analyzing Trends of Violence Against Civilians in Africa
Final project work for IDCE-296 Advanced Vector GIS. Clark University Spring 2024.

# Introduction

#### The Dataset
This project is an exploration of the civilian fatalities [dataset](https://acleddata.com/curated-data-files/#regional
) maintained by the Armed Conflict Location & Event Data Project (ACLED). [ACLED](https://acleddata.com/) is a "non-profit, non-governmental organization" and the "leading source of real-time data on political violence and protest activity around the world." Theirs is widely considered the most comprehensive dataset on global conflict, and is regularly cited by major news outlets such as the New York Times. The complete dataset on global civilian fatalities goes back to 1997 for some countries, and contains, as of March 2024, when we downloaded it, information on more than 327,000 incidents all over the world.  

#### Research Objective
The goal with this project is to analyze how hotspots of violence against civilians have shifted on the African continent over time, and explore if/how the nature of anti-civilian violence has changed. This will be accomplished through data analysis and visualization in the form of graphs and hot spot mapping.

# Preliminary Data Exploration
To begin with we'll do some exploration of the entire dataset, which includes every country in the world. We'll do this with Python, mostly using pandas and geopandas. See FatalitiesAnalysis.ipynb. The ACLED dataset is updated continuously, and the dataset used here was downloaded on March 22, 2024.

After aggregating total fatalities by country into a new dataframe, we will add a "start_year" field, taken from this handy [chart](https://acleddata.com/acleddatanew/wp-content/uploads/dlm_uploads/2019/01/ACLED_Country-and-Time-Period-coverage_updatedFeb2022.pdf), which allows us to also calculate a 'years' field, and a 'count_per_year' field, which represents the **average number of civilian fatalities per country per year**. 

```python
# calculate number of years of data collection for each country
fatalities['years'] = current_year - fatalities['Start Year']
# use this to get count per year
fatalities['count_per_year'] = fatalities['fatalities'] / fatalities['years']
# Clean up a couple fields
fatalities['start_year'] = fatalities['Start Year']  # Change 'Start Year' to 'start_year'
fatalities.drop(columns=['Country', 'Start Year'], inplace=True)  # Drop 'Country' column
# Sort the DataFrame by 'count_per_year' in descending order
fatalities = fatalities.sort_values(by='count_per_year', ascending=False)
# Selecting the top 15 rows
top_30_fatalities = fatalities.nlargest(30, 'count_per_year')
top_30_fatalities = top_30_fatalities.reset_index(drop=True)
top_30_fatalities
```

Here are the top 30, ranked by average number of civilian fatalities per year:

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/b1d41b79-2e69-4451-aa9f-9745807226a4)

I suspect some readers may find this chart surprising. Are people aware of the level of violence in Mexico, or Brazil?

The lack of parity between countries in how long data has been collected presents an issue when analyzing this data.
Every country in mainland Africa goes back to 1997, Latin America starts in 2018, the rest of the world is a number of different dates.

For example-- Syria started in 2017. How much higher would the rate of civilian fatalities be there if it had started a couple years earlier, like most of the other countries in the Middle East, and included the bulk of their Civil War?

But we'll poke around a bit more. This snippet adds a cumulative fatalities by country field to the dataset:
```python
# Create cumulative fatalities field
df.loc[:, 'event_date'] = pd.to_datetime(df['event_date'])
# Calculate cumulative fatalities
df['cumFatalities'] = df.groupby('country')['fatalities'].cumsum()
```

Which allows us to make this plot, cumulative fatalities in the 15 countries with the highest annual rate of civilian fatalities. This is another case study in why it's analytically challenging to compare countries with different data collection start points:
![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/3422ba88-e90b-47cc-b73f-22fcc715754d)

Some takeaways/thoughts on this chart:
- Is it a coincience that Nigeria and DRC were both quite peaceful for the same ~4 years in the mid 2000s?
- Iraq has generally stabilized
- Absolutely amazing and horrifying how consistent the rate of civilian death is in Mexico.
- At the risk of getting into politics-- ACLED goes through great pains to not include the deaths of combatants in this dataset. The point of the dataset is to estimate violence against civilians, not wartime combatant casualties. However there is one territory where they consider every death a civilian fatality. Even Hamas [admitted](https://www.reuters.com/world/middle-east/israels-six-week-drive-hit-hamas-rafah-scale-back-war-2024-02-19/) that they had lost 6000 fighters, as of mid February. It's odd, because ACLED otherwise takes them at their word regarding casualty numbers. Plus, the organization is well [aware](https://acleddata.com/knowledge-base/indirect-killing-of-civilians/) of the need to grapple with this issue. 
- The Darfur Genocide took place in Sudan from 2003-2005. It appears as though civilians were killed at roughly the same rate in Nigeria between 2014 and 2016 at the height of the Boko Haram insurgency.

Next I made a function to isolate and plot the steepest (deadliest) one year period for each country. 
A few examples:

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/d68c37f7-2a2a-463a-971c-c4c96f0b5e72)

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/3e113125-b9df-44e1-bfe4-804ddbb4575d)

# Africa by Region

From here on we will focus soley on Africa, analyzing spatial and temporal trends by country and region. 

To start with, we can look at **incidents** (not fatalities) by region:

![fatalities_year_region](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/881e30fd-6f60-43e8-86bc-8a2b1de8492e)

The number of incidents of violence against civilians in this dataset has absolutely exploded in the last decade or so. 
While this plot looks at **fatalities** by region over time:

![fatalities_region_line](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/85c5c126-1e3a-486b-a2bd-a008cdd56269)

They look quite different! And what is going on in Middle Africa and East Africa in the late 90s?

Based on those two graphs we can make a couple of general statements about the data:
- In the late 90s Middle Africa and Eastern Africa had relatively few incidents but very high numbers of fatalities
- West Africa has seen a sharp rise in number of fatalities in the last decade, though without an increase in the ratio of fatalities/per incident. 

These inferences are confirmed by calculating and plotting the average number of fatalities per incident by year, split by region.

![FatalitiesPer_ScatterPlot](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/3a2b2832-c831-4a93-afb9-6a6750880bd5)

The average number of fatalities per incident has trended down in pretty much every region, especially Middle Africa. This raises some questions about the data: Can this trend be explained largely by increased granularity of reporting? Could it be the case that in the past only significant incidents made it into the dataset? This seems quite likely. We may be able to answer this question by digging further.

We can also look at a comparison of incident count vs fatalities, by region:

![incidents_vs_fatalities_byRegion](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/5377da5f-5571-4b70-92ad-5fddec0c0fc9)

Every dot represents the number of fatalities vs incidents in a region for one year. Any line with a positive slope means that each incident in that region killed, on average, more than one person. Only in North Africa are there more incidents than fatalities. There are some significant outliers in Middle Africa in the top left quadrent of the graph. This we will have to investigate.

Going back to Python we can create a graph showing cumulative fatalities by year for each region.

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/4958b774-d610-499e-9caa-512a0efd0559)

I was surprised to see that even given the recent spike in West Africa, Eastern Africa still has more total fatalities. This is confirmed by a quick Summary Stats in ArcGIS Pro. 

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/52d11b9f-0d26-4639-b8c4-613e6bac9af8)

# Digging in to one Region: West Africa

There are a lot of different things that could be done with this data, depending on what question(s) you are trying to answer. West Africa is a region that I have some familiarity with, so we will take a closer look at the data for this region.
To do this we will first need to divide the allAfrica data by region:

![Screenshot 2024-04-18 235940](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/ad717c81-765a-4363-9c02-080e41145d51)

And plot fatalities per year by country. 

![WA_fatalities_year_country](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/acef14ba-78f9-4278-ab29-f5f5681a2ece)

Because I am somewhat familiar with West Africa, something catches my eye looking at this chart. Or rather, the lack of something. Where are the Liberian and Sierra Leon civil wars? The Sierra Leon Civil War lasted from 1991 to 2002, and the second Liberian Civil War lasted from 1999 to 2003. They don't really register on this plot. 
To be more sure we can view the data disentangled by country:

![WA_fatalities_country_grid](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/3cfa2aff-62e2-4742-aa8c-72f74d206912)

Barely even a blip is registered by either.
We can even pull out fatalities per year just for these two countries, to get precise numbers.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/2eb26c54-2c5c-4651-8093-9a9b21351713)

These numbers are inexplicably low. 
It takes less than a minute of internet searching to find information that suggests the data here is far from complete.

![Screenshot 2024-04-19 000844](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/cd1c5ba8-c02c-4e3b-aa5b-46ed05bb9801)

**Example 1: The seige of freetown, 1,000 civilian fatalities**

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/536b9108-d0cf-4f60-b07c-ad095aeff623)

**Example 2: 1999 Freetown Massacre, at least 3,500 civilian fatalities.**

These aren't obscure events, or footnotes. They are central moments in two major West African civil wars. And they are individual examples of what must be dozens or hundreds of incidents that are not in this dataset. Between those two civil wars the dataset is probably missing close to 100,000 civilian fatalities. And this is just an area I am vaguely familiar with. How many other areas are missing data from major conflicts?
All of which reinforces the fact that ACLED has a very ambitious mission, and despite the best of efforts the data will always be incomplete. Is it too incomplete to be useful? This may actually be the case for certain time periods in West Africa. There are regions of the world, as well as time periods in which it is very difficult to get accurate or complete information. 

However, the vastly increased number of reported incidents per year suggests that the dataset is getting better with time, and it is still almost certainly the best dataset on offer. 

## Incidents VS Fatalities

One thing I am curious about, at this stage, is the percentage of incidents in the dataset with 0 fatalities. The example of the Angolan Civil War, discussed in Section 2 would seem to suggest that earlier in the dataset only major casualty events are getting reported.
So we can calculate the percentage of incidents per year with 0 fatalities. I found this easier to do in Python:

```python
WA = df[df['region'] == 'Western Africa']
# Group the data by year and calculate the total number of entries for each year
yearly_totals = WA.groupby('year')['fatalities'].count()
# Group the data by year and calculate the number of entries with fatalities equal to 0 for each year
yearly_0 = WA[WA['fatalities'] == 0].groupby('year')['fatalities'].count()
# Calculate the percentage of entries with fatalities equal to 0 for each year
yearly_percent_0 = (yearly_0 / yearly_totals) * 100
# Print the resulting Series
yearly_percent_0
```
And we can plot it with a Pearson's coefficient.
![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/8e16a8c4-997a-4133-9b31-25d4ccfbd5e8)

A correlation of -0.70 suggest that either there is a higher threshold over time for what comprises/gets reported an incident, or incidents are becoming more more likely, on average, to lead to a civilian fatality. 

## Aggregating Data with Collect Events

Even confining our analysis to West Africa, we still have 26,000 incident points to analyze.
One way to go about visualizing all this data is with 'Collect Events', which will aggregate events by location

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/c5d16cc0-87b6-4bb4-aedb-ad40ce0f5019)

And allows us to create a map like this:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/977fc35e-1faa-4698-a148-0a8da6bc5746)

However, this probably leaves the viewer with more questions than answers regarding what exactly comprises a 'location.' The same neighborhood? The same city? Both, it turns out, according to the [Quick Guide to ACLED Data](https://acleddata.com/resources/quick-guide-to-acled-data/#s7):

"Locations are coded to named populated places, geostrategic locations, natural locations, or neighborhoods of larger cities. Geo-coordinates with four decimals are provided to assist in identifying and mapping named locations to a central point (i.e. a centroid coordinate) within that location. Geo-coordinates do not reflect a more precise location, like a block or street corner, within the named location."

## Aggregating Data by Administrative region

Perhaps a more effective way to represent this data is by aggregating at subnational administrative units, rather than single locations. 

We can pull a West Africa [Administrative Boundaries Level 1](https://data.humdata.org/dataset/west-and-central-africa-administrative-boundaries-levels) from [Humanitarian Data Exchange](https://data.humdata.org/). 
The ACLED dataset already has an 'admin1' field, which means we can simply aggregate this data, and then join it to the admin1 boundaries shapefile. 

The one tricky thing here is that there are some repeat names of admin1 zones across West Africa, so the aggregation and join must be done by both the admin1 and country field.
It could also have been accomplished with a spatial join.
For more details on how I did this see WestAfrica.ipynb, but here is the crux of it:

```python
fatalities_admin1 = WA.groupby(['country', 'admin1'])['fatalities'].sum().reset_index()
admin1_merged = WA_admin1.merge(fatalities_admin1, on=['country','admin1'], how='left')
incidents_admin1 = WA.groupby(['country', 'admin1']).size().reset_index(name='incident_count')# Print the summary
admin1_merged = admin1_merged.merge(incidents_admin1, on=['country','admin1'], how='left')
```

The result is a shapefile of admin1 boundries across West Africa with fields for total incidents and total fatalities in that region for the whole timeframe of the dataset. 

This data is a great use case for a bivariate color symbology, allowing us to quickly understand the ratio of fatalities to incidents by administrative region.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/7beaa386-91a5-49ad-9849-73eb5b780df0)

Unsurpisingly Northern Nigeria has the highest concentration of zones with both high incident numbers and high fatalities, and the Sahel has clusters of high incident activity too. A scatter plot of this data tells us basically the same thing:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/336b5ab7-b058-4cf4-84b7-909ecf1145a3)


