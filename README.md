<h1>E-Commerce Sales Analysis</h1>
<h2>Dataset: Superstore Sales Dataset (Kaggle) This is a classic real-world dataset with sales, profit, region, products, discounts, and customers.</h2>

``` python
#Importing all required libraries
import pandas as pd
import re
import matplotlib.pyplot as plt
#Reading CSV file
df = pd.read_csv("Sample_Superstore.csv", encoding="latin1")
```

<img width="1743" height="773" alt="ss1" src="https://github.com/user-attachments/assets/b32599fc-bf58-4a06-b710-945a9734775c" />

``` python
#Data Cleaning Process
#Standardizing Date
#Standardizing Date
def standardize_date(x):
    if pd.isna(x):
        return x

    x = str(x).strip()

       #If already in DD-MM-YYYY → keep as it is
    if re.fullmatch(r"\d{2}-\d{2}-\d{4}", x):
        return x

        #Try parsing other date formats (like 6/16/2016)
    try:
        d = pd.to_datetime(x, dayfirst=False, errors="raise")
        return d.strftime("%d-%m-%Y")
    except:
        return x    
        #keep original if not a valid date

   #Apply to column Oder Date
df["Order Date"] = df["Order Date"].apply(standardize_date)
df
```
<img width="1746" height="658" alt="image" src="https://github.com/user-attachments/assets/00999288-a7fb-463f-a21c-7e4660a53f20" />

```python
#Apply to column Ship date
df["Ship Date"] = df["Ship Date"].apply(standardize_date)
df.head()

```

<img width="1720" height="655" alt="image" src="https://github.com/user-attachments/assets/2b3a7ef8-c78d-48a5-9341-fadc9f6581b7" />

Exploratory Data Analysis
Which product categories generate the most revenue?

```python
category_sales = df.groupby("Category")["Sales"].sum()
bars=plt.barh(category_sales.index, category_sales.values)
plt.bar_label(bars)
plt.xlabel('Revenue')
plt.ylabel('Category')
plt.show()

```

<img width="857" height="544" alt="image" src="https://github.com/user-attachments/assets/124b1d93-9ecc-4006-8fb1-30ebf5a06d83" />

1.1 Which product sub-categories generate the most revenue? (Comprehending)

```python

Subcategory_sales = df.groupby("Sub-Category")["Sales"].sum()
bars=plt.barh(Subcategory_sales.index, Subcategory_sales.values)
plt.bar_label(bars)
plt.xlabel('Revenue')
plt.ylabel('Sub Category')
plt.show()

```

<img width="817" height="542" alt="image" src="https://github.com/user-attachments/assets/38fa59a4-cec0-4d6d-b17f-070375a6f38f" />

2 Which states/regions are most profitable and which are loss-making?

```python

region_profit = df.groupby("Region")["Profit"].sum()
bars = plt.barh(region_profit.index, region_profit.values)
plt.bar_label(bars)
plt.xlabel('Profit')
plt.ylabel('Region')

plt.show()

```

<img width="776" height="548" alt="image" src="https://github.com/user-attachments/assets/aa0c75e4-81aa-4c2a-a843-991ce75214c0" />

2.1 Region wise profit margin?

```python

region_margin = (
    df.groupby("Region")[["Sales", "Profit"]]
      .sum()
      .assign(Profit_Margin=lambda x: x["Profit"] / x["Sales"])
)
bars = plt.barh(region_margin.index,
    region_margin["Profit_Margin"])
plt.bar_label(bars)
plt.xlabel('Profit Margin')
plt.ylabel('Region')

plt.show()

```

<img width="794" height="569" alt="image" src="https://github.com/user-attachments/assets/a7b80abb-acbb-4d2d-8dd3-66ad17b80856" />

3 Does discount rate affect profit negatively or positively?

```python
  # 1. Convert Discount to Percentage
df["Discount_Percent"] = df["Discount"] * 100

  # 2. Create discount bands in percentage
discount_bins = [0, 10, 20, 30, 100]
discount_labels = ["0–10%", "10–20%", "20–30%", "30%+"]

df["Discount_Band"] = pd.cut(
    df["Discount_Percent"],
    bins=discount_bins,
    labels=discount_labels,
    include_lowest=True
)

  # 3. Calculate average profit per discount band
discount_profit_summary = (
    df.groupby("Discount_Band", observed=True)["Profit"]
      .mean()
      .reset_index()
)

print(discount_profit_summary)

 # 4. Scatter plot: Discount % vs Profit
plt.figure(figsize=(8, 5))
plt.scatter(df["Discount_Percent"], df["Profit"], alpha=0.4)
plt.xlabel("Discount (%)")
plt.ylabel("Profit")
plt.title("Impact of Discount Percentage on Profit")
plt.axhline(0, linestyle="--")   # profit/loss reference
plt.show()
```
<img width="947" height="708" alt="image" src="https://github.com/user-attachments/assets/52979c04-f260-4822-b990-929e432b0557" />

4 Which are the top 10 customers based on total sales?

```python

  # Example grouped data
region_profit = df.groupby("Customer Name")["Profit"].sum()

 # Sort and take top 10
top10 = region_profit.sort_values(ascending=False).head(10)

 # Plot bar chart
bars = plt.barh(top10.index, top10.values)

plt.bar_label(bars)
plt.ylabel("Names")
plt.xlabel("Profit")
plt.title("Top 10 Customers by Profit")
plt.figure(figsize=(50, 6))
plt.show()

```

<img width="908" height="566" alt="image" src="https://github.com/user-attachments/assets/fcd9d941-210f-4a8a-a070-68b7a6819083" />

4.1 Profit Margine per customer?

```python

 # 1. Calculate profit margin per customer
profit_margin = (
    df.groupby("Customer Name")[["Sales", "Profit"]]
      .sum()
      .assign(Profit_Margin=lambda x: x["Profit"] / x["Sales"])
)

  #  2. Select top 10 customers by profit margin
top10 = (
    profit_margin
    .sort_values(by="Profit_Margin", ascending=False)
    .head(10)
)

  #  3. Horizontal bar chart
plt.figure(figsize=(9, 6))
bars = plt.barh(top10.index, top10["Profit_Margin"])
plt.bar_label(bars, fmt="%.2f")
plt.xlabel("Profit Margin")
plt.ylabel("Customer Name")
plt.title("Top 10 Customers by Profit Margin")
plt.gca().invert_yaxis()    #highest at top
plt.show()

```
<img width="1144" height="697" alt="image" src="https://github.com/user-attachments/assets/b3bb0572-e5ac-4063-8920-7f3ade518a3d" />

Identifying Oldest Order date for further analysis

```python

  #  Convert to datetime (DD-MM-YYYY)
def Convert_datetime() :
     return pd.to_datetime(df["Order Date"], format="%d-%m-%Y", errors="coerce")
df["Order Date"] = Convert_datetime()
  #  Find minimum (oldest) date
min_date = df["Order Date"].min()
min_date = pd.to_datetime(min_date)
min_date = min_date.strftime("%d-%m-%Y")
min_date

```
Output : '02-01-2014'

Identifying Latest Order Date for further analysis

```python

  #  Find Maximum (oldest) date
  #  Convert to datetime (DD-MM-YYYY)
df["Order Date"] =Convert_datetime()
max_date = df["Order Date"].max()
max_date = pd.to_datetime(max_date)
max_date = max_date.strftime("%d-%m-%Y")
max_date

```
Output : '30-12-2017'

Are there any seasonal trends in sales over months/quarters?

5.1 Monthly trend for 2014

```python 

  #  Convert Order Date to datetime
df["Order Date"] = Convert_datetime()

  #  Filter data for year 2014
df_2014 = df[df["Order Date"].dt.year == 2014]

  #  Monthly sales aggregation
monthly_sales = (
    df_2014.groupby(df_2014["Order Date"].dt.month)["Sales"].sum()
)

  #  Plot
plt.figure(figsize=(10, 5))
plt.plot(monthly_sales.index, monthly_sales.values, marker="o")

plt.xlabel("Month")
plt.ylabel("Sales")
plt.title("Monthly Sales Trend – 2014")
plt.xticks(range(1, 13))     #Jan to Dec

plt.show()

```
<img width="1100" height="594" alt="image" src="https://github.com/user-attachments/assets/466da0d6-bdf1-42e1-8f3d-a39733163913" />

5.2 Monthly Trend for 2015

```python

  #  Convert Order Date to datetime
df["Order Date"] = Convert_datetime()

  #  Filter data for year 2014
df_2015 = df[df["Order Date"].dt.year == 2015]

 #   Monthly sales aggregation
monthly_sales = (
    df_2015.groupby(df_2015["Order Date"].dt.month)["Sales"].sum()
)

 #   Plot
plt.figure(figsize=(10, 5))
plt.plot(monthly_sales.index, monthly_sales.values, marker="o")

plt.xlabel("Month")
plt.ylabel("Sales")
plt.title("Monthly Sales Trend – 2015")
plt.xticks(range(1, 13))     #Jan to Dec

plt.show()

```

<img width="1090" height="595" alt="image" src="https://github.com/user-attachments/assets/f2a035c3-028a-497d-b868-f90d01fcb8f1" />

5.3 Monthly Trend for 2016

```python

   # Convert Order Date to datetime
df["Order Date"] = Convert_datetime()

  #  Filter data for year 2014
df_2016 = df[df["Order Date"].dt.year == 2016]

  #  Monthly sales aggregation
monthly_sales = (
    df_2016.groupby(df_2016["Order Date"].dt.month)["Sales"].sum()
)

  #  Plot
plt.figure(figsize=(10, 5))
plt.plot(monthly_sales.index, monthly_sales.values, marker="o")

plt.xlabel("Month")
plt.ylabel("Sales")
plt.title("Monthly Sales Trend – 2016")
plt.xticks(range(1, 13))     #Jan to Dec

plt.show()

```

<img width="1084" height="586" alt="image" src="https://github.com/user-attachments/assets/4e7da553-34d8-46cd-9a40-2164ed5e23b1" />

5.4 Monthly Trend for 2017

```python

  #  Convert Order Date to datetime
df["Order Date"] = Convert_datetime()

  #  Filter data for year 2017
df_2017 = df[df["Order Date"].dt.year == 2017]

  #  Monthly sales aggregation
monthly_sales = (
    df_2017.groupby(df_2017["Order Date"].dt.month)["Sales"].sum()
)

  #  Plot
plt.figure(figsize=(10, 5))
plt.plot(monthly_sales.index, monthly_sales.values, marker="o")

plt.xlabel("Month")
plt.ylabel("Sales")
plt.title("Monthly Sales Trend – 2017")
plt.xticks(range(1, 13))     #Jan to Dec

plt.show()

```

<img width="1115" height="595" alt="image" src="https://github.com/user-attachments/assets/4ade08ec-53f6-4118-a3f2-120ea65dd197" />

Adding quarter column to Dataframe

```python
df["Quarter"] = df["Order Date"].dt.to_period("Q")
df.groupby("Quarter")["Sales"].sum()
df

```

5.a Quarterly Trend for 2014

```python

  #  Add Quarter column
df_2014.loc[:, "Quarter"] = df_2014["Order Date"].dt.to_period("Q")


  #  Quarterly sales aggregation
quarterly_sales = (
    df_2014.groupby("Quarter")["Sales"].sum()
)

  #  Plot
plt.figure(figsize=(8, 5))
plt.plot(
    quarterly_sales.index.astype(str),
    quarterly_sales.values,
    marker="o"
)

plt.xlabel("Quarter")
plt.ylabel("Sales")
plt.title("Quarterly Sales Trend – 2014")

plt.show()

```
<img width="951" height="586" alt="image" src="https://github.com/user-attachments/assets/6c2bd89f-cf79-4b57-992e-d754ca75dd10" />

5.b Quarterly Trend for 2015

```python

  #  Add Quarter column
df_2015.loc[:, "Quarter"] = df_2015["Order Date"].dt.to_period("Q")


 #   Quarterly sales aggregation
quarterly_sales_2015 = (
    df_2015.groupby("Quarter")["Sales"].sum()
)

 #   Plot
plt.figure(figsize=(8, 5))
plt.plot(
    quarterly_sales_2015.index.astype(str),
    quarterly_sales_2015.values,
    marker="o"
)

plt.xlabel("Quarter")
plt.ylabel("Sales")
plt.title("Quarterly Sales Trend – 2015")

plt.show()

```
<img width="944" height="575" alt="image" src="https://github.com/user-attachments/assets/7a3461ea-9132-47fa-a7c1-e0b596ae47ea" />

5.c Quarterly Trend for 2016

```python

 #   Add Quarter column
df_2016.loc[:, "Quarter"] = df_2016["Order Date"].dt.to_period("Q")


  #  Quarterly sales aggregation
quarterly_sales_2016 = (
    df_2016.groupby("Quarter")["Sales"].sum()
)

 #   Plot
plt.figure(figsize=(8, 5))
plt.plot(
    quarterly_sales_2016.index.astype(str),
    quarterly_sales_2016.values,
    marker="o"
)

plt.xlabel("Quarter")
plt.ylabel("Sales")
plt.title("Quarterly Sales Trend – 2016")

plt.show()

```

<img width="939" height="585" alt="image" src="https://github.com/user-attachments/assets/744e5980-7ad0-4279-ab04-9d997e7cc99d" />

5.d Quarterly Trend for 2017

```python

  #Add Quarter column
df_2017.loc[:, "Quarter"] = df_2017["Order Date"].dt.to_period("Q")


  #Quarterly sales aggregation
quarterly_sales_2017 = (
    df_2017.groupby("Quarter")["Sales"].sum()
)

  #Plot
plt.figure(figsize=(8, 5))
plt.plot(
    quarterly_sales_2017.index.astype(str),
    quarterly_sales_2017.values,
    marker="o"
)

plt.xlabel("Quarter")
plt.ylabel("Sales")
plt.title("Quarterly Sales Trend – 2017")

plt.show()

```

<img width="916" height="562" alt="image" src="https://github.com/user-attachments/assets/c4ad9750-7ee5-4bb8-93e9-3c81aaa37632" />

Yearly Sales Trend

```python

 #Ensure Order Date is datetime
df["Order Date"] =Convert_datetime()

  #Aggregate sales year-wise
yearly_sales = (
    df.groupby(df["Order Date"].dt.year)["Sales"]
      .sum()
      .sort_index()
)

  #Plot line chart
plt.figure(figsize=(10, 5))
plt.plot(yearly_sales.index, yearly_sales.values, marker="o")

plt.xlabel("Year")
plt.ylabel("Total Sales")
plt.title("Yearly Sales Trend")
plt.xticks(yearly_sales.index)

plt.show()

```

<img width="1112" height="574" alt="image" src="https://github.com/user-attachments/assets/6ce1a03e-af66-46f6-87c6-6aa0721d9fb1" />


Reccomendations

<img width="1368" height="659" alt="image" src="https://github.com/user-attachments/assets/9aee86e6-cd5b-4b73-8d5d-ea65fed1eb08" />
