Importing the Libraries
!pip install chart_studio
Collecting chart_studio
  Downloading chart_studio-1.1.0-py3-none-any.whl (64 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 64.4/64.4 kB 476.0 kB/s eta 0:00:00
Requirement already satisfied: retrying>=1.3.3 in /opt/conda/lib/python3.7/site-packages (from chart_studio) (1.3.3)
Requirement already satisfied: requests in /opt/conda/lib/python3.7/site-packages (from chart_studio) (2.28.1)
Requirement already satisfied: plotly in /opt/conda/lib/python3.7/site-packages (from chart_studio) (5.10.0)
Requirement already satisfied: six in /opt/conda/lib/python3.7/site-packages (from chart_studio) (1.15.0)
Requirement already satisfied: tenacity>=6.2.0 in /opt/conda/lib/python3.7/site-packages (from plotly->chart_studio) (8.0.1)
Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/lib/python3.7/site-packages (from requests->chart_studio) (2022.6.15)
Requirement already satisfied: charset-normalizer<3,>=2 in /opt/conda/lib/python3.7/site-packages (from requests->chart_studio) (2.1.0)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /opt/conda/lib/python3.7/site-packages (from requests->chart_studio) (1.26.11)
Requirement already satisfied: idna<4,>=2.5 in /opt/conda/lib/python3.7/site-packages (from requests->chart_studio) (3.3)
Installing collected packages: chart_studio
Successfully installed chart_studio-1.1.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
# data operation libraries
import numpy as np
import pandas as pd

# importing visualisation libraries
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

# for chloroplath plotting
import chart_studio.plotly as py
import plotly.graph_objs as go 
import plotly
import cufflinks as cf
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
init_notebook_mode(connected=True)
cf.go_offline()

# for datetime operations
import datetime as dt

# pandas general settings
pd.options.display.max_columns = None
/opt/conda/lib/python3.7/site-packages/geopandas/_compat.py:115: UserWarning:

The Shapely GEOS version (3.9.1-CAPI-1.14.2) is incompatible with the GEOS version PyGEOS was compiled with (3.10.1-CAPI-1.16.0). Conversions between both will be slow.

Importing the Dataset
data = pd.read_csv('../input/global-super-store-dataset/Global_Superstore2.csv', encoding='windows-1252')
Data Preparation
data.head(2) #taking a look at the dataframe structure
Row ID	Order ID	Order Date	Ship Date	Ship Mode	Customer ID	Customer Name	Segment	City	State	Country	Postal Code	Market	Region	Product ID	Category	Sub-Category	Product Name	Sales	Quantity	Discount	Profit	Shipping Cost	Order Priority
0	32298	CA-2012-124891	31-07-2012	31-07-2012	Same Day	RH-19495	Rick Hansen	Consumer	New York City	New York	United States	10024.0	US	East	TEC-AC-10003033	Technology	Accessories	Plantronics CS510 - Over-the-Head monaural Wir...	2309.650	7	0.0	762.1845	933.57	Critical
1	26341	IN-2013-77878	05-02-2013	07-02-2013	Second Class	JR-16210	Justin Ritter	Corporate	Wollongong	New South Wales	Australia	NaN	APAC	Oceania	FUR-CH-10003950	Furniture	Chairs	Novimex Executive Leather Armchair, Black	3709.395	9	0.1	-288.7650	923.63	Critical
# correcting 'Order Date' variable
data[['order_day','order_month','order_year']] = data['Order Date'].str.split('-', expand=True)
data['Order Date'] = data['order_year'] + '/' + data['order_month'] + '/' + data['order_day']
data['Order Date'] = pd.to_datetime(data['Order Date'])
# doing likewise for 'Ship Date'
data[['ship_day','ship_month','ship_year']] = data['Ship Date'].str.split('-', expand=True)
data['Ship Date'] = data['ship_year'] + '/' + data['ship_month'] + '/' + data['ship_day']
data['Ship Date'] = pd.to_datetime(data['Ship Date'])
# dropping the support columns
data.drop(columns=['order_day','order_month','order_year','ship_day','ship_month','ship_year'], inplace=True)
data.info() #checkout the data types/ null rows and memory consumption
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 51290 entries, 0 to 51289
Data columns (total 24 columns):
 #   Column          Non-Null Count  Dtype         
---  ------          --------------  -----         
 0   Row ID          51290 non-null  int64         
 1   Order ID        51290 non-null  object        
 2   Order Date      51290 non-null  datetime64[ns]
 3   Ship Date       51290 non-null  datetime64[ns]
 4   Ship Mode       51290 non-null  object        
 5   Customer ID     51290 non-null  object        
 6   Customer Name   51290 non-null  object        
 7   Segment         51290 non-null  object        
 8   City            51290 non-null  object        
 9   State           51290 non-null  object        
 10  Country         51290 non-null  object        
 11  Postal Code     9994 non-null   float64       
 12  Market          51290 non-null  object        
 13  Region          51290 non-null  object        
 14  Product ID      51290 non-null  object        
 15  Category        51290 non-null  object        
 16  Sub-Category    51290 non-null  object        
 17  Product Name    51290 non-null  object        
 18  Sales           51290 non-null  float64       
 19  Quantity        51290 non-null  int64         
 20  Discount        51290 non-null  float64       
 21  Profit          51290 non-null  float64       
 22  Shipping Cost   51290 non-null  float64       
 23  Order Priority  51290 non-null  object        
dtypes: datetime64[ns](2), float64(5), int64(2), object(15)
memory usage: 9.4+ MB
# let's check out the columns which are suitable category column type

data.nunique()
Row ID            51290
Order ID          25035
Order Date         1430
Ship Date          1464
Ship Mode             4
Customer ID        1590
Customer Name       795
Segment               3
City               3636
State              1094
Country             147
Postal Code         631
Market                7
Region               13
Product ID        10292
Category              3
Sub-Category         17
Product Name       3788
Sales             22995
Quantity             14
Discount             27
Profit            24575
Shipping Cost     10037
Order Priority        4
dtype: int64
data['Ship Mode'] = data['Ship Mode'].astype('category')
data['Segment'] = data['Segment'].astype('category')
data['Country'] = data['Country'].astype('category')
data['Market'] = data['Market'].astype('category')
data['Region'] = data['Region'].astype('category')
data['Category'] = data['Category'].astype('category')
data['Sub-Category'] = data['Sub-Category'].astype('category')
data['Order Priority'] = data['Order Priority'].astype('category')
data.info() #check the reduction in memory consumption
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 51290 entries, 0 to 51289
Data columns (total 24 columns):
 #   Column          Non-Null Count  Dtype         
---  ------          --------------  -----         
 0   Row ID          51290 non-null  int64         
 1   Order ID        51290 non-null  object        
 2   Order Date      51290 non-null  datetime64[ns]
 3   Ship Date       51290 non-null  datetime64[ns]
 4   Ship Mode       51290 non-null  category      
 5   Customer ID     51290 non-null  object        
 6   Customer Name   51290 non-null  object        
 7   Segment         51290 non-null  category      
 8   City            51290 non-null  object        
 9   State           51290 non-null  object        
 10  Country         51290 non-null  category      
 11  Postal Code     9994 non-null   float64       
 12  Market          51290 non-null  category      
 13  Region          51290 non-null  category      
 14  Product ID      51290 non-null  object        
 15  Category        51290 non-null  category      
 16  Sub-Category    51290 non-null  category      
 17  Product Name    51290 non-null  object        
 18  Sales           51290 non-null  float64       
 19  Quantity        51290 non-null  int64         
 20  Discount        51290 non-null  float64       
 21  Profit          51290 non-null  float64       
 22  Shipping Cost   51290 non-null  float64       
 23  Order Priority  51290 non-null  category      
dtypes: category(8), datetime64[ns](2), float64(5), int64(2), object(7)
memory usage: 6.7+ MB
# making sure neither of our category columns have leading spaces

def remove_leading_spaces(df):
    for cols in df.columns:
        if df[cols].dtypes in ['object','category']:
            df[cols] = df[cols].str.strip()
        return df
data = remove_leading_spaces(data)
data.head(2)
Row ID	Order ID	Order Date	Ship Date	Ship Mode	Customer ID	Customer Name	Segment	City	State	Country	Postal Code	Market	Region	Product ID	Category	Sub-Category	Product Name	Sales	Quantity	Discount	Profit	Shipping Cost	Order Priority
0	32298	CA-2012-124891	2012-07-31	2012-07-31	Same Day	RH-19495	Rick Hansen	Consumer	New York City	New York	United States	10024.0	US	East	TEC-AC-10003033	Technology	Accessories	Plantronics CS510 - Over-the-Head monaural Wir...	2309.650	7	0.0	762.1845	933.57	Critical
1	26341	IN-2013-77878	2013-02-05	2013-02-07	Second Class	JR-16210	Justin Ritter	Corporate	Wollongong	New South Wales	Australia	NaN	APAC	Oceania	FUR-CH-10003950	Furniture	Chairs	Novimex Executive Leather Armchair, Black	3709.395	9	0.1	-288.7650	923.63	Critical
# generating years from our 'Order_year' variable because we are going 
# to need this in future analysis

data['Order_year'] = data['Order Date'].dt.year
# also total unique customer count is something we need in our future analysis

print('Number of unique customers made purchase in 2011: {}'.format(data[data['Order_year']==2011]['Customer Name'].nunique()))
print('Number of unique customers made purchase in 2012: {}'.format(data[data['Order_year']==2012]['Customer Name'].nunique()))
print('Number of unique customers made purchase in 2013: {}'.format(data[data['Order_year']==2013]['Customer Name'].nunique()))
print('Number of unique customers made purchase in 2014: {}'.format(data[data['Order_year']==2014]['Customer Name'].nunique()))
Number of unique customers made purchase in 2011: 795
Number of unique customers made purchase in 2012: 795
Number of unique customers made purchase in 2013: 795
Number of unique customers made purchase in 2014: 794
def total_purchase_in_year(row):
    Order_year = row[24]
    
    if Order_year in [2011,2012,2013]:
        return 795
    else:
        return 794
    
    
# generating  'unique_customers_within_year' based on associated year value
# for that particular row

data['unique_customers_within_year'] = data.apply(total_purchase_in_year, axis='columns')
Before generating revenue column let's understand the intution behing Revenue.

Revenue is another word for the amount of money a company generates from its sales.

Revenue is most simply calculated as the number of units sold multiplied by the selling price.



# Generating 'Revenue' column
data['Revenue'] = data['Sales'] * data['Quantity']
EDA
Solving the questions we have been asked

Customers Analysis
Question: 1Profile the customers based on their frequency of purchase - calculate frequency of purchase for each customer

Question: 2Do the high frequent customers are contributing more revenue

Question: 3Are they also profitable - what is the profit margin across the buckets

Question: 4Which customer segment is most profitable in each year.

Question: 5How the customers are distributed across the countries- -

Question:1
Profile the customers based on their frequency of purchase - calculate frequency of purchase for each customer

purchase_frequency = data.groupby(['Order_year','Customer Name'])
purchase_frequency.agg({'Customer Name': 'count',
                       'unique_customers_within_year': 'min',
                       'Revenue': 'sum',
                       'Profit': 'sum'}) 
Customer Name	unique_customers_within_year	Revenue	Profit
Order_year	Customer Name				
2011	Aaron Bergman	14	795	2693.78200	189.26450
Aaron Hawkins	15	795	47418.92150	1528.25570
Aaron Smayling	8	795	12117.70000	180.54020
Adam Bellavance	6	795	9210.63210	370.65270
Adam Hart	19	795	25909.57552	322.34912
...	...	...	...	...	...
2014	Xylona Preis	13	794	13240.71500	210.67240
Yana Sorensen	20	794	47494.72200	2175.74250
Yoseph Carroll	13	794	27667.53400	549.72100
Zuschuss Carroll	28	794	24306.65600	960.92900
Zuschuss Donatelli	14	794	8581.31970	468.34770
3179 rows × 4 columns

analysis_result = purchase_frequency.agg({'Customer Name': 'count',
                       'unique_customers_within_year': 'min',
                       'Revenue': 'sum',
                       'Profit': 'sum'})
analysis_result.rename(mapper={'Customer Name': 'Purchase_during_year'}, axis=1, inplace=True)
Calculating Customer Purchase Frequency

The repeat purchase rate is a calculation that shows you the percentage of your current customer base that has purchased at least a second time in a specific duration (usally take 365 days). This metric is influenced by your customer retention efforts and is a good indicator of the value you are providing your customers.



analysis_result['Customer_purchase_frequency'] = analysis_result['Purchase_during_year']/analysis_result['unique_customers_within_year'] *100
Answer:
Here we are only supposed to find the purchase frequency of each customer and not the one who are having the highest purchase frequency. So here is the result:

analysis_result.head(5)
Purchase_during_year	unique_customers_within_year	Revenue	Profit	Customer_purchase_frequency
Order_year	Customer Name					
2011	Aaron Bergman	14	795	2693.78200	189.26450	1.761006
Aaron Hawkins	15	795	47418.92150	1528.25570	1.886792
Aaron Smayling	8	795	12117.70000	180.54020	1.006289
Adam Bellavance	6	795	9210.63210	370.65270	0.754717
Adam Hart	19	795	25909.57552	322.34912	2.389937
Question:2
Do the high frequent customers are contributing more revenue?

The question here is comapring the high purchase frequency customers with high revenue generating customers. In the previous question we found out the purchase frequency of each customer, so out of those we will finf out highest purchse frequency customers for that year and then will compare to the highest revenue generator for that year.

tmp_df = analysis_result.reset_index()
tmp_df.head()
Order_year	Customer Name	Purchase_during_year	unique_customers_within_year	Revenue	Profit	Customer_purchase_frequency
0	2011	Aaron Bergman	14	795	2693.78200	189.26450	1.761006
1	2011	Aaron Hawkins	15	795	47418.92150	1528.25570	1.886792
2	2011	Aaron Smayling	8	795	12117.70000	180.54020	1.006289
3	2011	Adam Bellavance	6	795	9210.63210	370.65270	0.754717
4	2011	Adam Hart	19	795	25909.57552	322.34912	2.389937
grouped_object = tmp_df.groupby(['Order_year'])
freq_df = pd.DataFrame(columns=tmp_df.columns)
for g,d in grouped_object:
    highest_freq_customers = d.nlargest(1, 'Customer_purchase_frequency')
    freq_df = pd.concat([freq_df, highest_freq_customers])
def highlight_cols(x): 
    df = x.copy()
    df.loc[:, ['Customer Name','Customer_purchase_frequency']] = 'background-color: green'
    df[['Order_year','Purchase_during_year','unique_customers_within_year','Revenue','Profit']] = 'background-color: grey'
    return df 
display(freq_df.style.apply(highlight_cols, axis = None))
 	Order_year	Customer Name	Purchase_during_year	unique_customers_within_year	Revenue	Profit	Customer_purchase_frequency
210	2011	David Philippe	31	795	37504.175040	247.489920	3.899371
1433	2012	Rob Dowd	42	795	41813.193100	1482.860300	5.283019
2194	2013	Pete Kriz	47	795	33062.639000	328.075600	5.911950
3075	2014	Shahid Collister	49	794	74150.276580	1636.835020	6.171285
rev_df = pd.DataFrame(columns=tmp_df.columns)
for g,d in grouped_object:
    highest_rev_customers = d.nlargest(1, 'Revenue')
    rev_df = pd.concat([rev_df, highest_rev_customers])
def highlight_cols(x): 
    df = x.copy()
    df.loc[:, ['Customer Name','Revenue']] = 'background-color: green'
    df[['Order_year','Purchase_during_year','unique_customers_within_year','Profit','Customer_purchase_frequency']] = 'background-color: grey'
    return df 
display(rev_df.style.apply(highlight_cols, axis = None))
 	Order_year	Customer Name	Purchase_during_year	unique_customers_within_year	Revenue	Profit	Customer_purchase_frequency
687	2011	Sean Miller	15	795	158047.509000	-901.745700	1.886792
1481	2012	Sean Christensen	21	795	115865.898000	994.162000	2.641509
1596	2013	Adrian Barton	15	795	136078.958000	4843.389200	1.886792
3142	2014	Tom Ashbrook	23	794	140379.010000	5371.299700	2.896725
Answer:
We can clearly see by comparing both the tables that neither of the high puchase frequency customers are there in the high revenue generating customer taable.

So the answer is no, high purchase frequency customers aren't contributing to high revenue.

Question 3
Are they also profitable - what is the profit margin across the buckets

profit_df = pd.DataFrame(columns=tmp_df.columns)
for g, d in grouped_object:
    highest_profit = d.nlargest(1, 'Profit')
    profit_df = pd.concat([profit_df,highest_profit])
def highlight_cols(x): 
    df = x.copy()
    df.loc[:, ['Customer Name','Profit']] = 'background-color: green'
    df[['Order_year','Purchase_during_year','unique_customers_within_year','Revenue','Customer_purchase_frequency']] = 'background-color: grey'
    return df 
display(profit_df.style.apply(highlight_cols, axis = None))
 	Order_year	Customer Name	Purchase_during_year	unique_customers_within_year	Revenue	Profit	Customer_purchase_frequency
672	2011	Sanjit Chand	18	795	76791.853000	5733.372000	2.264151
1337	2012	Mike Gockenbach	13	795	88443.941600	4839.418200	1.635220
2321	2013	Tamara Chand	23	795	109272.976000	8536.494300	2.893082
3007	2014	Raymond Buch	22	794	70156.140000	7444.534300	2.770781
Answer:
From the table above we can see that neither of the customers who were in high purchase frequency table or high revenue table are here in the high profitable customer table. May be these are the customer who are purchasing low quantity but the profit margin is higher on their purchase.

Question 4:
Which customer segment is most profitable in each year.

segment_group = data.groupby(['Order_year','Segment'])
high_profit_df = segment_group.agg({'Profit':'sum'}).unstack()
high_profit_df.style.background_gradient(cmap='Spectral', subset=pd.IndexSlice[:, pd.IndexSlice[:,'Consumer']])
 	Profit
Segment	Consumer	Corporate	Home Office
Order_year	 	 	 
2011	117337.494060	84746.935740	46856.381740
2012	165799.190940	90556.699920	51059.388240
2013	208427.733980	125707.939080	72799.557120
2014	257675.363080	140196.753920	106293.853460
Answer:
We can see that every year consumer segment is triggering more profit to the firm.

Question: 4
How the customers are distributed across the countries?

country_group = data.groupby(['Country'])
customer_distribution = country_group.agg({'Customer ID':'count'})
customer_distribution.columns = ['Customer_count']
customer_distribution.reset_index(inplace=True)
customer_distribution
Country	Customer_count
0	Afghanistan	55
1	Albania	16
2	Algeria	196
3	Angola	122
4	Argentina	390
...	...	...
142	Venezuela	194
143	Vietnam	265
144	Yemen	30
145	Zambia	102
146	Zimbabwe	80
147 rows × 2 columns

country_map = dict(type='choropleth',
           locations=customer_distribution['Country'],
           locationmode='country names',
           z=customer_distribution['Customer_count'],
            reversescale = True,
           text=customer_distribution['Country'],
           colorscale='earth',
           colorbar={'title':'Customer Count'})
layout = dict(title='Customer Distribution over Countries',
             geo=dict(showframe=False,projection={'type':'mercator'}))
Answer:
choromap = go.Figure(data = [country_map],layout = layout)
iplot(choromap)
Product Analysis
Question: 1Which country has top sales?

Question: 2Which are the top 5 profit-making product types on a yearly basis

Question: 3How is the product price varying with sales - Is there any increase in sales with the decrease in price at a day level

Question: 4What is the average delivery time across the counties - bar plot

Question: 1
Which country has top sales?

country_group = data.groupby('Country')
country_sales = country_group.agg({'Sales':'sum'})
country_sales.sort_values(by='Sales', ascending=False)
Sales
Country	
United States	2.297201e+06
Australia	9.252359e+05
France	8.589311e+05
China	7.005620e+05
Germany	6.288400e+05
...	...
Tajikistan	2.427840e+02
Macedonia	2.096400e+02
Eritrea	1.877400e+02
Armenia	1.567500e+02
Equatorial Guinea	1.505100e+02
147 rows × 1 columns

Answer:
We can see that United States has top sales. Things are better when they are visually presented. Let's plot top 10 sales countries.

import squarify
top_10_sales = country_sales.nlargest(10, 'Sales')
top_10_sales.index
CategoricalIndex(['United States', 'Australia', 'France', 'China', 'Germany',
                  'Mexico', 'India', 'United Kingdom', 'Indonesia', 'Brazil'],
                 categories=['Afghanistan', 'Albania', 'Algeria', 'Angola', 'Argentina', 'Armenia', 'Australia', 'Austria', ...], ordered=False, dtype='category', name='Country')
plt.figure(figsize=(15,7))
revs = top_10_sales['Sales'].values
labels = ['United States: 2297200.8603',
         'Australia: 925235.853',
         'France: 858931.083',
         'China: 700562.025',
         'Germany: 628840.0305',
         'Mexico: 622590.61752',
         'India: 589650.105',
         'United Kingdom: 528576.3',
         'Indonesia: 404887.4979',
         'Brazil: 361106.41896']
squarify.plot(revs, label=labels,color= sns.color_palette('copper'), alpha=0.7)
plt.show()

Question 2:
Which are the top 5 profit-making product types on a yearly basis

year_category_group = data.groupby(['Order_year','Sub-Category'])
year_category_proft_df = year_category_group.agg({'Profit':'sum'})
year_category_proft_df
Profit
Order_year	Sub-Category	
2011	Accessories	15719.8606
Appliances	22838.4413
Art	10399.0233
Binders	11447.2053
Bookcases	27518.8575
...	...	...
2014	Paper	20975.8306
Phones	70657.6413
Storage	39016.9521
Supplies	7365.4090
Tables	-30545.9084
68 rows × 1 columns

year_category_proft_df.reset_index(inplace=True)
category_yearly_profit = year_category_proft_df.groupby('Order_year')
top5_profit_category = pd.DataFrame(columns=year_category_proft_df.columns)
for g, d in category_yearly_profit:
    high_profit_categories = d.nlargest(5, 'Profit')
    top5_profit_category = pd.concat([top5_profit_category,high_profit_categories])
Answer:
Below dataframe includes top 5 profit making products for each year.

top5_profit_category.style.background_gradient(cmap='Spectral', subset=pd.IndexSlice[:, 'Profit'])
 	Order_year	Sub-Category	Profit
13	2011	Phones	53927.489500
6	2011	Copiers	30375.093440
5	2011	Chairs	29943.157100
4	2011	Bookcases	27518.857500
1	2011	Appliances	22838.441300
23	2012	Copiers	51843.227600
30	2012	Phones	45223.049800
17	2012	Accessories	33507.100200
22	2012	Chairs	28755.346700
21	2012	Bookcases	28137.267100
40	2013	Copiers	72300.691180
47	2013	Phones	46908.825200
38	2013	Bookcases	43049.244400
35	2013	Appliances	41485.516000
39	2013	Chairs	40449.492100
57	2014	Copiers	104048.535960
64	2014	Phones	70657.641300
55	2014	Bookcases	63219.050500
52	2014	Appliances	53040.500500
51	2014	Accessories	41593.928600
Question 3:
How is the product price varying with sales - Is there any increase in sales with the decrease in price at a day level

Note: This question could have been more specifir to a coutry/product category and for a specific year but all we have asked is to see the trend between sales and price. Let's try to plot it as asked.

How to Calculate Unit Price. The unit price can be found using a simple formula if the quantity and total cost is known. Simply divide the total price by the quantity to find the unit price. Thus, the unit price is equal to the total price divided by the quantity.

data['Unit_price'] = data['Sales']/data['Quantity']
data['Order_day'] = data['Order Date'].dt.day
Answer:
From below chart we can see that when the prices are lower sales are high and when the prices increase, sales decrease.

g5 = sns.FacetGrid(data, row = 'Order_day', col = 'Order_year', hue = 'Order_day')
kwe = dict(s = 50, linewidth = 0.5, edgecolor = 'black')
g5 = g5.map(plt.scatter, 'Unit_price', 'Sales')
g5.set(xlim=(0,100), ylim=(0,100))
for ax in g5.axes.flat:
    ax.plot((0,100),(0,100), c = 'gray', ls = '--')
g5.add_legend()
<seaborn.axisgrid.FacetGrid at 0x7f2db1aa0810>

Question: 4
What is the average delivery time across the counties - bar plot

data['Delivery_duration'] = data['Ship Date']-data['Order Date']
country_group = data.groupby('Country')
delivery_duration_df = country_group.agg({'Delivery_duration':'mean'})
delivery_duration_df['Duration_in_hours'] = delivery_duration_df['Delivery_duration'] / dt.timedelta(hours=1)
We have 147 unique countries in our dataframe. Plotting them altogether on a same plot wouln't be possible. So I will only plot top 10 sales countries.

delivery_duration_df
Delivery_duration	Duration_in_hours
Country		
Afghanistan	4 days 11:46:54.545454545	107.781818
Albania	3 days 15:00:00	87.000000
Algeria	3 days 20:26:56.326530612	92.448980
Angola	4 days 04:55:04.918032786	100.918033
Argentina	3 days 19:48:55.384615384	91.815385
...	...	...
Venezuela	4 days 06:33:24.123711340	102.556701
Vietnam	3 days 19:28:18.113207547	91.471698
Yemen	4 days 00:00:00	96.000000
Zambia	3 days 21:52:56.470588235	93.882353
Zimbabwe	3 days 15:18:00	87.300000
147 rows × 2 columns

Answer:
top10_sales_country_DD =top_10_sales.merge(delivery_duration_df, how='left', left_index=True, right_index=True)
top10_sales_country_DD.reset_index(inplace=True)
top10_sales_country_DD.sort_values(by='Duration_in_hours')
# we can see that China and Brazil are providing fastest deliveries
Country	Sales	Delivery_duration	Duration_in_hours
3	China	7.005620e+05	3 days 21:22:12.765957446	93.370213
9	Brazil	3.611064e+05	3 days 21:46:43.001876172	93.778612
2	France	8.589311e+05	3 days 22:39:31.135479306	94.658649
1	Australia	9.252359e+05	3 days 22:39:48.156503348	94.663377
0	United States	2.297201e+06	3 days 23:00:46.828096858	95.013008
5	Mexico	6.225906e+05	3 days 23:18:03.812405446	95.301059
6	India	5.896501e+05	3 days 23:22:57.491961414	95.382637
8	Indonesia	4.048875e+05	3 days 23:36:10.359712230	95.602878
4	Germany	6.288400e+05	4 days 01:45:59.709443099	97.766586
7	United Kingdom	5.285763e+05	4 days 02:29:54.488671157	98.498469
top10_sales_country_DD.iplot(kind='bar',x='Country', y='Duration_in_hours',
                            title= 'Countries And their Average Product Delivery Duration in Hours',
                            xTitle='Countries',
                            yTitle= 'AVG Delivery Duration in hours')
