# Analyzing trends of violence against civiliance in Africa
Final project work for IDCE-296 Advanced Vector GIS. Clark University Spring 2024.

This project is an exploration of the civilian fatalities [dataset](https://acleddata.com/curated-data-files/#regional
) maintained by the Armed Conflict Location & Event Data Project (ACLED)
. 
[ACLED](https://acleddata.com/) is a "non-profit, non-governmental organization" and the "leading source of real-time data on political violence and protest activity around the world."

The ultimate goal with this project is to analyze how hotspots of violence against civilians have shifted on the continent over time. 

To begin wtih we'll just do some exploration of the entire dataset, which includes every country in the world, in python, mostly using pandas and geopandas, see CivilianAnalysis.ipynb. The ACLED dataset is updated continuously. The dataset used here was downloaded on March 22, 2024.

We start by adding a "start_year" field, taken from this handy [chart](https://acleddata.com/acleddatanew/wp-content/uploads/dlm_uploads/2019/01/ACLED_Country-and-Time-Period-coverage_updatedFeb2022.pdf
), which allows us to calculate the **average number of civilian fatalities per country per year**. 

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

But we'll still poke around. This snippet adds a cumulative fatalities by country field to the dataset:
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

Then I also made a function to pull out and plot the steepest (deadliest) one year period for each country. A few examples

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/d68c37f7-2a2a-463a-971c-c4c96f0b5e72)

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/3e113125-b9df-44e1-bfe4-804ddbb4575d)


## Bringing the data into ArcGIS Pro to Analyze Trends in Africa

From here on out we will focus soley on Africa, analyzing spatial and temporal trends by country, and region. 

To start with, we can look at **incidents** (not fatalities) by region:

![fatalities_year_region](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/881e30fd-6f60-43e8-86bc-8a2b1de8492e)

While this plot does look at fatalities by region over time:

![fatalities_region_line](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/85c5c126-1e3a-486b-a2bd-a008cdd56269)

They look quite different! And what is going on in Middle Africa and East Africa in the late 90s?

Based on those two graphs we can make a couple of general statements about the data:
- In the late 90s Middle Africa and Eastern Africa had several incidents with very high numbers of fatalities
- West Africa has seen a steep rise in number of fatalities in the last decade, though with a relatively low fatality rate/incident. 

These inferences are confirmed by comparing incident count to fatalities, by region:

![incidents_vs_fatalities_byRegion](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/5377da5f-5571-4b70-92ad-5fddec0c0fc9)

Every dot represents the number of fatalities vs incidents in a region for one year. Any line with a positive slope means that each incident in that region killed, on average, more than one person. Only in North Africa are there more incidents than fatalities. There are some significant outliers in Middle Africa in the top left quadrent of the graph. This we will have to investigate.

Going back to Python we can create a graph showing cumulative fatalities by year for each region.

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/4958b774-d610-499e-9caa-512a0efd0559)

I was surprised to see thate even given the recent spike in Africa, Middle Africa still has more total fatalities. This is confirmed by a quick summary stats in ArcGIS Pro. 

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/52d11b9f-0d26-4639-b8c4-613e6bac9af8)


