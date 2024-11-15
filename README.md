# CognoRise-Infotech-BigMart

# BIGMART SALES ANALYSIS REPORT

## TABLE OF CONTENT 
- [INTRODUCTION](#INTRODUCTION)
- [KPI (Key Performance Indicator)](#KPI-(Key-Performance-Indicator))
- [SQL](#SQL)
- [ANALYSIS](#ANALYSIS)
- [RECOMMENDATION](#RECOMMENDATION)
- [CONCLUSION](#CONCLUSION)
- [RECOMMENDATION](#RECOMMENDATION)
- [REVIEW](#REVIEW)
  


### INTRODUCTION

BigMart Sales Data provides a detailed view of retail operations, including item specifics, outlet details, and sales labels. This dataset empowers businesses to strategically tailor product modifications and marketing efforts by understanding customer preferences and optimizing resources based on outlet types and sales trends. 
With insights into diverse factors such as outlet size, location, and establishment year, businesses can make informed decisions to enhance customer engagement and maximize the impact of their retail strategies. 

### KPI (Key Performance Intellegence)

1.	Top product sold by the retailer inorder to know the desire of the customer base on its year of establishment.
2.	How does it affects the customer health base on the fat content and how the sales of each product increase base on this.
3.	How do fix price and the MPR affects the outlet of a particular product.
4.	Which product has the highest sales base on the ranking 
5.	How does the year of establishment affect the sales of the store.
6.	The most visible item to the customer.
7.	How does the outlet size affect the sales of items.
8.	Store which has the highest product sales

## SQL 

This is the code use for [DATA CLEANING]

```SQL
-- <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<BIGMART SALES PROJECT>>>>>>>>>>>>>>>>>>>>>>>

/* AIMS AND OBJECTIVE 
	DATA CLEANING
		A. Creating database BIGMART_DB
			i. creating tables(train, test, train_1, test_2)
		B. DATA WRANGLING Involving
			data wrangling: adjust by modifying the column datatype and the standardization of data seem
			to be okay so no standardization is needed to be done
        C. Feauture engineering
			i. Removing duplicate 
            ii. Adding extra column to help remove duplicates
            iii. Filling blank spaces
            IV. Trimming to remve white space */

-- dropping any bigmart database if exist and creating database a new database

Drop database if exists BigMart_DB;
create database BigMart_DB;

-- dropping and creating a new table train 
-- import data from external source to the databasre table

drop table if exists train;
create table Train(
	Item_identifier varchar(6),
    item_weight float(10,5),
    Item_fat_content varchar(16),
    Item_visibility decimal(15,12),
    Item_type varchar(16),
    Item_mpr decimal(15,12),
    Outlet_identifier varchar(16),
    Outlet_establishment_year INT,
    Outlet_size varchar(11),
    Outlet_location_type varchar(8),
    Outlet_type varchar(20),
    item_outlet_sales decimal(15,7)
);

-- select all from train table
select 
	*
from 
	Train;

-- dropping and creating a new table test
-- import data from external source to the database table

drop table if exists test;
create table test(
	Item_identifier varchar(6),
    item_weight float(10,5),
    Item_fat_content varchar(16),
    Item_visibility decimal(15,12),
    Item_type varchar(16),
    Item_mpr decimal(15,12),
    Outlet_identifier varchar(16),
    Outlet_establishment_year INT,
    Outlet_size varchar(11),
    Outlet_location_type varchar(8),
    Outlet_type varchar(20)
);

-- select all from test table

select 
	*
from 
	Test;
 
-- self joinning table to join two unequal train and test table which is not possible

select 
	t1.*,
    t2.*
from train t1
cross join 
train t2 on t1.item_identifier = t2.item_identifier;

-- Removing duplicate value by adding a new column to the table using window function
select 
	*,
    row_number() over (partition by item_identifier,item_weight,item_fat_content,item_visibility,item_type) as row_num
from 
	train;
    
drop table if exists train_1;

-- creating a new table from previously existing table to add window function and remove duplicate to clean up the table

create table train_1(
	Item_identifier varchar(6),
    item_weight float(10,5),
    Item_fat_content varchar(16),
    Item_visibility decimal(15,12),
    Item_type varchar(16),
    Item_mpr decimal(15,12),
    Outlet_identifier varchar(16),
    Outlet_establishment_year INT,
    Outlet_size varchar(11),
    Outlet_location_type varchar(8),
    Outlet_type varchar(20),
    item_outlet_sales decimal(15,7) ,
    row_num int
);

insert into train_1
select 
	*,
    row_number() over (partition by item_identifier,item_weight,item_fat_content,item_visibility,item_type) as row_num
from 
	train;
    
select 
	*
from 
	train_1;
select 
	*
from 
	train_1
where row_num > 1;

Delete
from 
	train_1
where row_num > 1;

-- checking for any null value
select 
	count(*)
from 
	train_1
where outlet_size = "High";

select 
	count(*)
from 
	train_1
where outlet_size = "small";

select 
	count(*)
from 
	train_1
where outlet_size = "medium";

select 
	*
from
	train_1
where item_type = "soft drinks" and outlet_size = "small";


-- changing blank spaces to null
select 
	*
from
	train_1
where outlet_size = "";

update train_1
set outlet_size = null
where outlet_size = "";

-- selecting null values and fiding solutions to it

select 
	*
from
	train_1
where outlet_size is null;

select
	*
from
	train_1 
where item_type = "snack foods";

-- triming the entire table to remove white spaces 

update train_1
set Item_identifier = trim(Item_identifier);

update train_1
set item_weight = trim(item_weight);

update train_1
set Item_fat_content = trim(Item_fat_content);

update train_1
set Item_visibility = trim(Item_visibility);

update train_1
set Item_type = trim(Item_type);

update train_1
set Item_mpr = trim(Item_mpr);

update train_1
set Outlet_identifier = trim(Outlet_identifier);

update train_1
set Outlet_establishment_year = trim(Outlet_establishment_year);

update train_1
set Outlet_size = trim(Outlet_size);

update train_1
set Outlet_location_type = trim(Outlet_location_type);

update train_1
set Outlet_type = trim(Outlet_type);

update train_1
set item_outlet_sales = trim(item_outlet_sales);

-- self joining table to fill up blank spaces
select 
	t1.*,
    t2.*
from train_1 t1
join
train_1 t2 on t1.item_identifier = t2.item_identifier
where t1.Outlet_size is not null and t2.Outlet_size is null;

update train_1 t1
join train_1 t2
	on t1.item_identifier = t2.item_identifier
set t2.outlet_size = t1.outlet_size
where t1.outlet_size is not null and t2.outlet_size is null;

-- selecting and deleting untrusted data which is 0.6% of the entire data

select 
	*
from 
	train_1
where Outlet_size is null;

delete 
from
	train_1
where Outlet_size is null;

alter table train_1
drop column row_num;

select 
	*
from 
	train_1;

drop table if exists test_2;
-- creating a new table from previously existing table to add window function and remove duplicate to clean up the table

CREATE TABLE `test_2` (
  `Item_identifier` varchar(6) DEFAULT NULL,
  `item_weight` float(10,5) DEFAULT NULL,
  `Item_fat_content` varchar(16) DEFAULT NULL,
  `Item_visibility` decimal(15,12) DEFAULT NULL,
  `Item_type` varchar(16) DEFAULT NULL,
  `Item_mpr` decimal(15,12) DEFAULT NULL,
  `Outlet_identifier` varchar(16) DEFAULT NULL,
  `Outlet_establishment_year` int DEFAULT NULL,
  `Outlet_size` varchar(11) DEFAULT NULL,
  `Outlet_location_type` varchar(8) DEFAULT NULL,
  `Outlet_type` varchar(20) DEFAULT NULL,
	`row_num` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- selecting all from test_2 table
select 
	*
from 
	Test_2;
    
insert into test_2
select 
	*,
    row_number() over (partition by item_identifier,item_weight,item_fat_content,item_visibility,item_type) as row_num
from 
	test;

select 
	*
from 
	test_2
where row_num > 1;

Delete
from 
	test_2
where row_num > 1;

-- triming the entire table to remove white spaces 

update test_2
set Item_identifier = trim(Item_identifier);

update test_2
set item_weight = trim(item_weight);

update test_2
set Item_fat_content = trim(Item_fat_content);

update test_2
set Item_visibility = trim(Item_visibility);

update test_2
set Item_type = trim(Item_type);

update test_2
set Item_mpr = trim(Item_mpr);

update test_2
set Outlet_identifier = trim(Outlet_identifier);

update test_2
set Outlet_establishment_year = trim(Outlet_establishment_year);

update test_2
set Outlet_size = trim(Outlet_size);

update test_2
set Outlet_location_type = trim(Outlet_location_type);

update test_2
set Outlet_type = trim(Outlet_type);

-- checking for null values

select 
	*
from
	test_2
where outlet_size = "";

-- changing blank space to null

update test_2
set outlet_size = null
where outlet_size = "";

-- selecting all null values
select 
	*
from 
	test_2
where Outlet_size is null;

-- self joining table to fill up blank spaces

select 
	t1.*,
    t2.*
from test_2 t1
join
test_2 t2 on t1.item_identifier = t2.item_identifier
where t1.Outlet_size is not null and t2.Outlet_size is null;

update test_2 t1
join test_2 t2
	on t1.item_identifier = t2.item_identifier
set t2.outlet_size = t1.outlet_size
where t1.outlet_size is not null and t2.outlet_size is null;

-- selecting and deleting untrusted data which is 0.6% of the entire data

delete 
from
	test_2
where Outlet_size is null;

select 
	*
from 
	Test_2;
-- deleting row_num column
alter table test_2
drop column row_num;

SELECT
	*
FROM 
	test_2;
    
select 
	*
from 
	train_1;

-- <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<END>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

### ANALYSIS

This report highlights that among products sold at outlets established in 1999, snack foods emerged as the top-performing category. With total sales reaching $700,000, snack foods demonstrated the strongest sales performance in this segment. This finding underscores the popularity and demand for snack foods in these particular outlets, offering valuable insights for strategic planning in product stocking, marketing efforts, and resource allocation to further capitalize on consumer preferences in established outlets.

![big1](https://github.com/user-attachments/assets/8a87ea13-2b7a-4a3e-b2f4-df2cb87132b0)


This report reveals that seafood had the lowest overall sales, totaling $52,629, at outlets established in 1997. Despite this, seafood still recorded the highest sales among products specifically available in outlets founded that year. This insight indicates a unique demand for seafood in 1997-established outlets, providing a basis for targeted product positioning and marketing strategies to improve sales in similar settings.

A strong customer preference for low-fat foods, reflecting a growing awareness of health and wellness. This trend is particularly impressive, as low-fat options are often associated with healthier dietary choices, aligning with consumer demand for products that support a balanced lifestyle.
In contrast, high-fat foods are linked to various health concerns, such as increased risk of cardiovascular disease, obesity, and other chronic conditions. High-fat diets can lead to elevated cholesterol levels and higher calorie intake, which may impact long-term health. The notable shift toward low-fat products suggests that customers are increasingly prioritizing foods that offer nutritional benefits without compromising their health, providing an opportunity for targeted marketing and product development in low-fat food options. Regular product too are doing well as well which brings about the growth of the store in terms of it sales.
Price plays a significant role in influencing product sales across different location tiers, with notable trends observed in small and medium-sized products. In Tier 2 location, small, affordable products achieve the highest sales, suggesting that customers in these areas are price-sensitive and favor lower-cost items. In Tier 1 location, while sales volumes are high, customers tend to purchase products in smaller quantities, indicating a preference for quality over quantity and a possible willingness to pay more for premium options. In Tier 3 location, medium-sized products see the highest sales, pointing to a balance between affordability and product size that appeals to customers in these areas. These trends demonstrate how 
tailoring product pricing and sizing to the purchasing behaviors of each location tier can effectively boost sales.

This report highlights that low-priced products achieve significantly higher visibility and sales compared to their high-priced counterparts. The affordability of these products makes them more appealing to a broader customer base, driving greater demand and accessibility. This trend suggests that price-sensitive consumers are more inclined toward purchasing lower-cost items, which directly impacts sales volume. For businesses, this insight underscores the importance of offering competitively priced products to maximize reach and capture a larger market share, especially in segments where visibility and volume are critical to sales performance.

![big2](https://github.com/user-attachments/assets/f923d280-15e1-41c6-9c03-da735715fed9)


## RECOMMENDATION


1.	Expand Low-Fat Product Offerings: Given the strong customer preference for low-fat foods, consider expanding the range of healthy, low-fat products. Emphasize their health benefits in marketing campaigns to further attract health-conscious consumers. Additionally, invest in clear labeling to highlight low-fat options on shelves, enhancing visibility and appeal.
2.	Optimize Product Pricing by Location Tier: Tailor pricing strategies to align with the purchasing behavior in each location tier:
o	Tier 1: Focus on premium-quality products in smaller quantities, as customers here are willing to pay for quality.
o	Tier 2: Prioritize small, low-priced products that align with the price sensitivity of these customers. Consider introducing value packs to boost sales further.
o	Tier 3: Position medium-sized products as the preferred option, balancing affordability and volume to appeal to this segment.
3.	Leverage High-Visibility, Low-Priced Products: Low-priced products have proven to generate high visibility and drive sales. Use this to the company’s advantage by strategically placing low-cost items in prominent store locations. Promotional offers, bulk discounts, or value packs can further enhance the appeal and increase foot traffic, especially in competitive markets.
4.	Capitalize on Location-Specific Product Preferences: Products like snack foods, which perform exceptionally well in certain outlet types, should be prominently featured in outlets with similar profiles. Utilize data-driven stocking and product placement strategies to maximize the performance of top-selling items by location.
5.	Enhance Marketing Efforts on High-Demand Products in Key Outlets: Based on establishment year insights, it’s clear that certain products have a strong performance in specific outlet types. Use targeted marketing efforts, such as in-store promotions and digital advertising, to reinforce product visibility and encourage sales in these key outlets.
6.	Educate Customers on the Benefits of Healthy Products: Since health is a growing priority, provide informational materials that educate customers on the health risks associated with high-fat products and the benefits of choosing low-fat alternatives. This can build brand loyalty and encourage repeat purchases among health-focused consumers.
7.	Invest in Data Analytics for Continuous Optimization: Continue to leverage data analytics to monitor and understand sales trends, customer preferences, and regional behaviors. Regular analysis of this data will allow the company to adjust product offerings, marketing strategies, and pricing models to stay aligned with evolving market demands.

## CONCLUSION

This is an accurate analysis and recommendation provided base on the available set of data. And by implementing these recommendations, the company can strengthen its competitive edge, cater to diverse customer needs across different markets, and position itself for sustained growth and expansion.

## REVIEW

Review the Tableau dashboard for more information and analysis

[DASHBOARD](https://public.tableau.com/app/profile/lekan.haruna/viz/BigMart_Sales_17315878755680/BigMart)

