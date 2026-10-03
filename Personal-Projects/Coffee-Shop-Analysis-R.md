# Coffee Shop Exploratory Analysis |  2024-04-12

> [!NOTE]
> For better visual, download the HTML file included in respiratory.

## Contents of Table
- Data Background
- Objectives
- Enviroment Setup
  - Connecting to database
  - Extracting Data
  - Data Cleaning
  - Missing Values
  - Feature Creation
- Exploratory Analysis
  - Univariate Analysis
     - Summary Statistics
     - Distributions
  - Bivariate & Multivariate Analysis
     - Numerical Variables Relationship
     - Sales Performance
     - Sales Performance By Location
     - Sales Performance By Category, Product, and Size
- Conclusion

# Data Background
The dataset comprises transactional records from Maven Roasters, a fictional NYC-based coffee shop operating across three distinct locations. It encompasses comprehensive details such as transaction dates, timestamps, geographical specifics, and product-level information.

Based on the dataset information, these are the columns and their description:
| Field | Type | Descriptions |
| --- | --- | --- |
| transaction_id | Numeric | Unique identifier for each transaction. |
| transaction_date | Date | Date when the transaction occurred. |
| transaction_time | Time	| Time of the transaction. |
| transaction_qty	| Numeric	| Quantity of products purchased in a transaction. |
| store_id	| Numeric | Unique identifier for each store location. |
| store_location	| Text	| Name or description of the store’s physical location. |
| product_id	| Numeric	| Unique identifier for each product sold. |
| unit_price	| Numeric	| Price of a single unit of the product in the transaction. |
| product_category	| Text	| General category to which the product belongs (e.g., Coffee, Tea, Drinking Chocolate). |
| product_type	| Text	| Specific type or variant of the product (e.g., Gourmet brewed coffee, Brewed Chai tea, Hot chocolate). |
| product_detail	| Text	| Additional details about the product(e.g., specific flavor, size, or blend) |

# Objectives
Our primary goal in this analysis is to understand the data and generate valuable insights for future decision-making. We will explore the data as much as possible. We can achieve our primary goal by performing the following:

1.) Identify the data structure: Through exploratory analysis, we can recognize the various data types that are present, how they are distributed, and the general structure of the data.

2.) Ensure data quality: We will evaluate the reliability and quality of the data by detecting anomalies and outliers in the dataset, addressing missing values, and correcting data type errors.

2.) Identify patterns and relationships: Analyze the dataset’s pattern and the relationship between its columns by examining it and applying different techniques to find hidden patterns.

# Enviroment Setup
```r
library(tidyverse) # General
library(lubridate) # Date Converter
library(tidycomm)  # Statistical Summary
library(scales)    # Numerical Labelling
library(e1071)     # Skew Scalling / Class Analysis
library(reshape2)  # Restructure and Aggregate Data
 
library(ggplot2)   # Data Visualization
library(ggthemes)  # Data Visual Theme
library(patchwork) # Combining Visuals

library(RPostgres) # PostgreSQL Database

 # General Color
 color = '#2879D8'
  
 # General Theme
 theme_set(theme_fivethirtyeight())
```

## Connecting to Database
```r
# Credentials 
warehouse <- config::get('datawarehouse')

# Create connection to database
con <- dbConnect(RPostgres::Postgres(),
                 dbname = warehouse$database,
                 host = warehouse$hostname,
                 port = warehouse$portid,
                 user = warehouse$username,
                 password = warehouse$password)
```

## Extracting Data
```r
# Extracting Data
data <- dbGetQuery(con, "SELECT * FROM coffee_shop")

# Disconnecting to Database
dbDisconnect(con)
```
Let’s take a quick look of the data
```r
##   transaction_id transaction_date transaction_time transaction_qty store_id store_location product_id unit_price   product_category product_type              product_detail
## 1              1       2023-01-01         07:06:11               2        5    1 Lower Manhattan         32        3.0             Coffee
## 2              2       2023-01-01         07:08:56               2        5    2 Lower Manhattan         57        3.1                Tea
## 3              3       2023-01-01         07:14:04               2        5     3 Lower Manhattan         59        4.5 Drinking Chocolate
## 4              4       2023-01-01         07:20:24               1        5     4 Lower Manhattan         22        2.0             Coffee
## 5              5       2023-01-01         07:22:41               2        5     5 Lower Manhattan         57        3.1                Tea

##            product_type              product_detail
## 1 Gourmet brewed coffee                 Ethiopia Rg
## 2       Brewed Chai tea    Spicy Eye Opener Chai Lg
## 3         Hot chocolate           Dark chocolate Lg
## 4           Drip coffee Our Old Time Diner Blend Sm
## 5       Brewed Chai tea    Spicy Eye Opener Chai Lg
```

## Data Cleaning
**Data Types**
```r
## 'data.frame':    149116 obs. of  11 variables:
##  $ transaction_id  : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ transaction_date: chr  "2023-01-01" "2023-01-01" "2023-01-01" "2023-01-01" ...
##  $ transaction_time: chr  "07:06:11" "07:08:56" "07:14:04" "07:20:24" ...
##  $ transaction_qty : int  2 2 2 1 2 1 1 2 1 2 ...
##  $ store_id        : int  5 5 5 5 5 5 5 5 5 5 ...
##  $ store_location  : chr  "Lower Manhattan" "Lower Manhattan" "Lower Manhattan" "Lower Manhattan" ...
##  $ product_id      : int  32 57 59 22 57 77 22 28 39 58 ...
##  $ unit_price      : num  3 3.1 4.5 2 3.1 3 2 2 4.25 3.5 ...
##  $ product_category: chr  "Coffee" "Tea" "Drinking Chocolate" "Coffee" ...
##  $ product_type    : chr  "Gourmet brewed coffee" "Brewed Chai tea" "Hot chocolate" "Drip coffee" ...
##  $ product_detail  : chr  "Ethiopia Rg" "Spicy Eye Opener Chai Lg" "Dark chocolate Lg" "Our Old Time Diner Blend Sm" ...
```
We will have several changes to our data types and columns to make our analysis easier:

- **transaction_id**: from integer to character since it’s a unique identifier for each transaction.

- **transaction_date**: from character to date it will make an unclear output if set to character.

- **transaction_time**: from character to timestamps it will make an unclear output if set to character.

- **store_id**: from integer to character since it’s a unique identifier for each store location.

- **product_id**: from integer to character since it’s a unique identifier for each product.

**Converting Data Types**
```r
# Transaction ID to character
data$transaction_id <- as.character(data$transaction_id)

# Transaction Date to Date
data$transaction_date <- ymd(data$transaction_date)

# Transaction Time to Datestamps
data$transaction_time <- hms(data$transaction_time)

# Store ID to character
data$store_id <- as.character(data$store_id)

# Product ID to character
data$product_id <- as.character(data$product_id)
```

Now that we have done converting columns, let's proceed to our next step.

## Missing Values
```r
##                  missing_count proportion
## transaction_id               0          0
## transaction_date             0          0
## transaction_time             0          0
## transaction_qty              0          0
## store_id                     0          0
## store_location               0          0
## product_id                   0          0
## unit_price                   0          0
## product_category             0          0
## product_type                 0          0
## product_detail               0          0
```

 As shown table above, we can see we have complete record, no missing values at all in our data which will lead us straight to our next step.

 ## Feature Creation
 **Revenue**
 We can create a new column from our existing columns by multiplying the quantity and price.
 ```r
# Adding Feature: Revenue
data$revenue <- (data$transaction_qty * data$unit_price)
```
**Size**
We can extract the size detail of every transactions from product details provided in the data and create column
```r
# Addding Feature: Size
data <- data %>% 
  mutate(size = ifelse(str_detect(product_detail, 'Rg') == TRUE, 'Regular', 
                       ifelse(str_detect(product_detail, 'Lg') == TRUE, 'Large', 
                              ifelse(str_detect(product_detail, 'Sm') == TRUE, 'Small', 'Undefined'))))
```
**Hour, Day of Week, and Month**
Extracting the day of the week, month, and year from the Transaction Date column that will help us to understand the transaction trend later on.
```r
# Adding feature: Hour
data$hour <- as.character(hour(data$transaction_time))

# Adding feature: Day of week
data$dayofweek <- as.character(wday(data$transaction_date))

# Adding feature: Month
data$month <- as.character(month(data$transaction_date))
```
**Removal of Columns**
Since some of the columns in our dataset are already represented in other formats, we will be removing those extra columns.
```r
# Removal of columns
data <- subset(data, select = -c(transaction_time, store_id, product_id, product_detail))
```
Let’s take a look at our dataset before continuing.
```r
## Rows: 149,116
## Columns: 12
## $ transaction_id   <chr> "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "1…
## $ transaction_date <date> 2023-01-01, 2023-01-01, 2023-01-01, 2023-01-01, 2023…
## $ transaction_qty  <int> 2, 2, 2, 1, 2, 1, 1, 2, 1, 2, 1, 2, 1, 1, 2, 2, 1, 1,…
## $ store_location   <chr> "Lower Manhattan", "Lower Manhattan", "Lower Manhatta…
## $ unit_price       <dbl> 3.00, 3.10, 4.50, 2.00, 3.10, 3.00, 2.00, 2.00, 4.25,…
## $ product_category <chr> "Coffee", "Tea", "Drinking Chocolate", "Coffee", "Tea…
## $ product_type     <chr> "Gourmet brewed coffee", "Brewed Chai tea", "Hot choc…
## $ revenue          <dbl> 6.00, 6.20, 9.00, 2.00, 6.20, 3.00, 2.00, 4.00, 4.25,…
## $ size             <chr> "Regular", "Large", "Large", "Small", "Large", "Undef…
## $ hour             <chr> "7", "7", "7", "7", "7", "7", "7", "7", "7", "7", "7"…
## $ dayofweek        <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
## $ month            <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
```
## Exploratory Analysis
### Univariate Analysis
#### Summary Statistics
**Numerical Variables Summary Statistics**
```r
##  transaction_date     transaction_qty   unit_price        revenue       
##  Min.   :2023-01-01   Min.   :1.000   Min.   : 0.800   Min.   :  0.800  
##  1st Qu.:2023-03-06   1st Qu.:1.000   1st Qu.: 2.500   1st Qu.:  3.000  
##  Median :2023-04-24   Median :1.000   Median : 3.000   Median :  3.750  
##  Mean   :2023-04-15   Mean   :1.438   Mean   : 3.382   Mean   :  4.686  
##  3rd Qu.:2023-05-30   3rd Qu.:2.000   3rd Qu.: 3.750   3rd Qu.:  6.000  
##  Max.   :2023-06-30   Max.   :8.000   Max.   :45.000   Max.   :360.000
```
**Insights:**
- **Date Range**: The dataset spans from January 1, 2023 to June 30, 2023.
- **Potential Outliers**: Revenue shows potential outliers with huge values.
- **Right-Skewed Distribution**: Quantity, and Revenue shows their mean are higher than median indicating possible right-skewed distribution.

**Categorical Variables Summary Statistics**
```r
## # A tibble: 8 × 6
##   Variable              N Missing Unique Mode            Mode_N
## * <chr>             <int>   <int>  <dbl> <chr>            <int>
## 1 transaction_id   149116       0 149116 1                    1
## 2 store_location   149116       0      3 Hell's Kitchen   50735
## 3 product_category 149116       0      9 Coffee           58416
## 4 product_type     149116       0     29 Brewed Chai tea  17183
## 5 size             149116       0      4 Regular          45789
## 6 hour             149116       0     15 10               18545
## 7 dayofweek        149116       0      7 6                21701
## 8 month            149116       0      6 6                35352
```
**Insights:**

- **Top Store Location**: Out of three locations, Hell’s Kitchens’ place is the most popular, accounting for 50,735 transactions.

- **Top Category**: Coffee was the most selling category accounting for 58,416 transactions.

- **Top Product**: Brewed Chai Tea was the most selling product with 17,183 sold product.

- **Top Size**: Regular size was the most frequent ordered size with 45,789 sold.

- **Busiest Month**: June has the most transactions, recording 35,352 transactions.

- **Busiest Day**: Friday was the popular day with 21,701 transactions.

- **Busiest Hour**: Around 10 a.m. was the busiest hour for the business, with 18,545 transactions took place during this period.

#### Distribution
**Numerical Variables Distribution**

![image](https://github.com/user-attachments/assets/43ea1c2e-97be-44bb-ba90-29a2aac9f717)

Due to outliers, it’s difficult to visualize the **interquartile range** of these columns: Price and Revenue. To better visualize it, we can temporarily zoom in on our y-axis.

![image](https://github.com/user-attachments/assets/2912ea91-1923-4150-a811-b282713da424)

**Insights:**

- **Quantity**: The quantity for transactions range from 1 to 2. There are outliers exceeding more than 4 quantity.
- **Price**: The price falls between 2.5 USD to 3.75 USD. Outliers exceeding above 5, and we have on exceptional reaching over 40.
- **Revenue**: The generated revenue from transaction falls between 3 USD to 6 USD. There are outliers above 10 USD, one reaching over 300 USD.

**Dealing with Outliers**
As noted earlier in summary statistics of numerical variables, all variables show potential outliers. Based on our boxplot, Revenue column reveals a massive value generated. This outliers in Revenue column are exceeding more than 300 USD, before we deal with this outliers, let’s investigate it first.
```r
##    transaction_id transaction_date transaction_qty store_location unit_price
## 1            9340       2023-01-17               8 Hell's Kitchen         45
## 2            9395       2023-01-17               8 Hell's Kitchen         45
## 3           68976       2023-04-17               8 Hell's Kitchen         45
## 4           69151       2023-04-17               8 Hell's Kitchen         45
## 5           98233       2023-05-17               8 Hell's Kitchen         45
## 6           98529       2023-05-17               8 Hell's Kitchen         45
## 7          133523       2023-06-17               8 Hell's Kitchen         45
## 8          133674       2023-06-17               8 Hell's Kitchen         45
## 9          133744       2023-06-17               8 Hell's Kitchen         45
## 10         149043       2023-06-30               8 Hell's Kitchen         45
##    product_category  product_type revenue      size hour dayofweek month
## 1      Coffee beans Premium Beans     360 Undefined    9         3     1
## 2      Coffee beans Premium Beans     360 Undefined    9         3     1
## 3      Coffee beans Premium Beans     360 Undefined    9         2     4
## 4      Coffee beans Premium Beans     360 Undefined   11         2     4
## 5      Coffee beans Premium Beans     360 Undefined    9         4     5
## 6      Coffee beans Premium Beans     360 Undefined   11         4     5
## 7      Coffee beans Premium Beans     360 Undefined    9         7     6
## 8      Coffee beans Premium Beans     360 Undefined   10         7     6
## 9      Coffee beans Premium Beans     360 Undefined   11         7     6
## 10     Coffee beans Premium Beans     360 Undefined   11         6     6
```
As we can see, the transactions are constant in terms of the quantity of units purchased, the store’s location, and the type or category of products purchased. Although we can presume that all of these transactions are either from the same customer or from different customers, but, due to insufficient information and proof, we can’t prove these assumptions.

Furthermore, we will have to remove these transactions from our data set to keep the consistency and accuracy for further analysis.

**Removing Outliers**
```r
data <- filter(data, revenue < 300)
```
Now that we removed the outliers, we will continue our analysis using Histograms

![image](https://github.com/user-attachments/assets/b1a6fd4e-d54a-4ed7-a11c-4b1bf99d9d3d)

As we mentioned earlier in summary statistics, all numerical variables are possible right-skewed distributed. We can confirm this by measuring their skewness.
```r
## [1] "Price Skewness: 8.40690038717276"
## [1] "Quantity Skewness: 0.69371272637651"
## [1] "Revenue Skewness: 5.0473896206454"
```
As shown above, we can see that all numerical variables are indeed right-skewed distributed, with Price column being the most noticeable.

**Categorical Variable Distribution**

![image](https://github.com/user-attachments/assets/91908457-abf1-4d05-9f92-a6f9247d9364)

**Insights:**

- **Product Category**: Coffee was the best selling product category with 58,416 transactions.

- **Product Type**: Brewed Chai Tea, with 17,183 transactions, was the most popular product type under the tea category.

![image](https://github.com/user-attachments/assets/e81c583a-9871-40c5-bf6f-0f913547cc81)

**Insights:**

- **Store Location**: Hell’s kitchen was the popular with more than 50,000 transactions. Astoria being the second, only few hundreds off with Hell’s kitchen.
- **Cup Size**: Regular was the most ordered cup size, large being the second. We can see that we have massive amount of undefined cup size with only a minimum Amount of difference to Large

![image](https://github.com/user-attachments/assets/cfa05a2c-8f7b-43b9-af86-05a7b139f273)

**Insights:**

- **Hourly**: The shop’s busiest hours were commonly in the morning based on the hourly distribution, which showed a pattern in customer behavior. 10 AM is the busiest time of day at the store, with over 18,000 transactions.

- **DayOfWeek**: A stable trend all throughout the week with an average of 20,000 transactions.

- **Month**: An increasing trend shown in Month distribution, as June was the highest with more than 30,000 transactions.

### Bivariate & Multivariate Analysis
**Numerical Variables Relationship**

![image](https://github.com/user-attachments/assets/f70679e1-5c98-4530-b73a-67d31b2b9bb7)

As shown above, we have high correlations between unit price and revenue, and low correlation between revenue and quantity.

Let’s check the numerical relationship using pair plot.

![image](https://github.com/user-attachments/assets/1420d8a5-2371-4bb4-892e-6efeedc3ee2d)

The strong positive correlation between Revenue and Unit Price is clear. The behavior of customers, who typically purchase higher-quality goods, makes this possible. We can presume that expensive coffee is a better product. Customers in this situation are more likely to spend money on high-quality goods, which boosts overall sales.

On the other hand, we can observe the low correlation between Quantity and Revenue. This is made possible by the fact that the coffee industry, where our dataset is derived from, typically values quality over quantity in consumer behavior. Only a tiny proportion of customers in the transaction purchased more than two coffees, as can be seen in our transaction quantity distribution.

**Sales Performance**

![image](https://github.com/user-attachments/assets/42ac8449-5a66-437a-9359-c6764f4bdff2)

The sales revenue show a strong increasing incline pattern from January to end of June as shown above, with June reaching the highest point in sales performance; with sales exceeding 6,000 USD.

If we thoroughly examine the trend line, we can see that there is a declining trend at the beginning of each month and a significant fall at the end of each month. April until the end of June is when it’s most noticeable. We can look into this unusual pattern in more detail.

![image](https://github.com/user-attachments/assets/414f13e3-f8e2-470a-99cd-b4156fda1390)

If we examine closely, the price trend line fluctuates. But quantity is the opposite. It displays the price flipped. The average quantity decreased as the average product price increased. We could assume that marketing techniques like sales, discounts, and the introduction of new or distinctive products drive consumers to purchase costly products.

Let’s now delve to daily average sales by week.

![image](https://github.com/user-attachments/assets/672a74a1-ca8a-4ea5-9532-4a19d60ee4e3)

The average daily sales were lowest on Sunday, with a sharp increase on Monday and a sharp decline on Tuesday, indicating an unusual pattern in the purchasing behavior of the customers. We can see that the trend line has a constant increase until Friday despite the initial dip. On Saturday, the daily average fell once more.

The weekday started with a sharp spike, which was followed by a steep decrease and a slowly increasing ascent until the end of the workday. This tendency may be related to the high volume of clients who are working during the workday. Customers that enjoy food and services and are searching for quick and convenient solutions. The sales are typically higher on weekdays.

![image](https://github.com/user-attachments/assets/baa9f41a-4358-4f72-833c-340a03de4db1)

The data shows that business hours begin at 6 in the morning. An early-morning steep curve suggests that most business sales occur more frequently during this time. From 8 a.m. up until 10 a.m. are the most active. As time goes by, business revenues begin to sharply fall, starting around 11 a.m. after which there was a short and constant decrease until the end of business hours.

**Sales Performance By Location**

![image](https://github.com/user-attachments/assets/d1b123f8-a6cd-4807-b19d-38ce7815536a)

As can be seen above, Hell’s Kitchen Store was ranked highest out of the three stores. There’s only minimal difference in revenue between these three establishments.

We can further analyze the data for each store if there’s an underlying pattern for consumer behavior. We can investigate this by analyzing the median quantity and price for every store.

![image](https://github.com/user-attachments/assets/dee71da8-71a9-44dc-86cd-558bbe1635af)

The median quantity and price purchases give the same outcome in every store. This indicates that regardless of the different store locations, the quantity and price purchased is constant. Given the consistency of the median quantity throughout the stores, this provides no significant context for our analysis.

NOTE: The median was used instead of the mean. The data has outliers that could potentially influence the result of the mean and give unrealiable insights.

**Sales Performance by Category, Product, and Size**

First, let’s determine the number of category this coffee shop offers.

```r
## [1] "Total Category: 9"
```

![image](https://github.com/user-attachments/assets/e5fdc985-a840-4d01-87fe-3d4aceb91b5c)

With sales of roughly 250,000 USD, coffee was the leading category. Tea came in second with approximately 195,000 USD in sales. There is a noticeable difference in margin between tea and the bakery category.

We can analyze the trend of these categories average daily sales for a 6-month period. Providing an insight into how these categories evolve over time. 

![image](https://github.com/user-attachments/assets/24b0260e-bdbc-45c1-a0ac-36a8005177fb)

When it comes to average daily sales, the category of coffee beans constantly leads the trend. It’s noticeable that this category fluctuates, particularly on day seventeen, assuming this category is caused by sales or promotional activity.

The majority of the categories’ daily average sales—bread, coffee, drinking chocolate, loose tea, tea, and flavors—show consistency. If we notice, several products’ sales consistently start on the 2nd week of each month. It’s possible that the business conducts restocking every first week of the month.

This insight is connected to consumer purchasing behavior, in which there was a fluctuating trend in the daily average product and quantity purchased. These categories—branded and coffee beans—make significant contributions to shifts or fluctuations in patterns. 

Let’s look at products. But first, let’s determine the number of products this coffee shop offers.

```r
## [1] "Total Products: 29"
```

The coffee shop offers 29 unique products, let’s continue

![image](https://github.com/user-attachments/assets/def43545-5243-4065-9e3d-85ba7648a048)

These are the best-selling products out of the coffee shop’s 29 distinctive products. With sales of more than 80,000 USD, Barista Espresso, which falls within the coffee category. Then came Gourmet Brewed Coffee, Hot Chocolate, and Brewed Chai Tea, all of which had sales exceeding $65,000 USD.

![image](https://github.com/user-attachments/assets/97fada4a-c900-4716-9786-7e5ef1fd9aee)

Standing at the top of the list with sales of over 250,000 USD was Large, followed by undefined at exceed 210,000 USD and regular at approximately 200,000 USD. Assuming that small sizes are unpopular, we can see that small sizes only had small sales.

The undefined size belongs to another category, such as food, branded, or custom-sized cups made from other products.

# Conclusion
In summary, the company has grown impressively, with overall revenue of 698,812 USD increasing steadily over the previous half-year. Another important factor contributing to fluctuations in sales, especially in the middle of the month is the introduction of premium and branded products. Additionally, consumers show a preference for high-quality products, which increases sales for the business. Interesting trends can be seen in the analysis of sales patterns, with Monday standing out as the busiest day in the early morning hours between 8 a.m. up until 10 a.m. having the largest number of customers. Revenue differs minimally between branches, despite these fluctuations. The category of coffee has the lead, accounting for 38.6% of total sales. Barista Espresso generates the most revenue, but brewed chai tea is the most popular in terms of sales quantity. Lastly, it appears that customers prefer large and regular cup sizes.




