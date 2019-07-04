```
add jar hdfs:/user/training/json-serde-1.3.8-jar-with-dependencies.jar;
add jar hdfs:/user/training/json-udf-1.3.8-jar-with-dependencies.jar;

add jar json-serde-1.3.8-jar-with-dependencies.jar;
add jar json-udf-1.3.8-jar-with-dependencies.jar;

CREATE EXTERNAL TABLE test.business4 (
address string,
business_id string,
categories array<string>,
city string,
hours struct<friday:string, monday:string, saturday:string, sunday:string, thursday:string,
tuesday:string, wednesday:string>,
is_open int,
latitude double,
longitude double,
name string,
neighborhood string,
postal_code string,
review_count int,
stars double,
state string,
Attributes struct<
Accepts_Insurance:boolean,
Ages_Allowed:string,
Alcohol:string,
Bike_Parking:boolean,
Business_Accepts_Bitcoin:boolean,
Business_Accepts_Credit_Cards:boolean,
By_Appointment_Only:boolean,
Byob:boolean,
BYOB_Corkage:string,
Caters:boolean,
Coat_Check:boolean,
Corkage:boolean,
Dogs_Allowed:boolean,
Drive_Thru:boolean,
Good_For_Dancing:boolean,
Good_For_Kids:boolean,
Happy_Hour:boolean,
Has_TV:boolean,
Noise_Level:string,
Open24Hours:boolean,
Outdoor_Seating:boolean,
Restaurants_Attire:string,
Restaurants_Counter_Service:boolean,
Restaurants_Delivery:boolean,
Restaurants_Good_For_Groups:boolean,
Restaurants_Reservations:boolean,
Restaurants_Table_Service:boolean,
Restaurants_Take_Out:boolean,
Smoking:string,
WheelchairAccessible:boolean,
WiFi:string,
Ambience:struct<
Casual:boolean,
Classy:boolean,
Divey:boolean,
Hipster:boolean,
Intimate:boolean,
Romantic:boolean,
Touristy:boolean,
Trendy:boolean,
Upscale:boolean>,
BestNights:struct<
Friday1:boolean,
Monday1:boolean,
Saturday1:boolean,
Sunday1:boolean,
Thursday1:boolean,
Tuesday1:boolean,
Wednesday1:boolean>,
BusinessParking:struct<
Garage:boolean,
Lot:boolean,
Street:boolean,
Valet:boolean,
Validated:boolean>,
DietaryRestrictions:struct<
Dairy_Free:boolean,
Gluten_Free:boolean,
Halal:boolean,
Kosher:boolean,
Soy_Free:boolean,
Vegan:boolean,
Vegetarian:boolean>,
GoodForMeal:struct<
Breakfast:boolean,
Brunch:boolean,
Dessert:boolean,
Dinner:boolean,
Latenight:boolean,
Lunch:boolean>,
HairSpecializesIn:struct<
Africanamerican:boolean,
Asian:boolean,
Coloring:boolean,
Curly:boolean,
Extensions:boolean,
Kids:boolean,
Perms:boolean,
Straightperms:boolean>,
Music:struct<
BackgroundMusic:boolean,
Dj:boolean,
Jukebox:boolean,
Karaoke:boolean,
Live:boolean,
NoMusic:boolean,
Video:boolean>,
restaurantpricerange2:int>)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/business';


CREATE TABLE test.biz4
STORED AS PARQUET
AS SELECT * FROM test.business4;

CREATE TABLE test.exploded
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/business/exploded'
AS SELECT * FROM test.business4 LATERAL VIEW explode(categories) c AS cat_exploded;

CREATE TABLE test.restaurants
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/mboldin/yelp/business/restaurants'
AS
SELECT * FROM test.exploded WHERE cat_exploded="Restaurants";

CREATE EXTERNAL TABLE test.review (
business_id string,
cool int,
review_date string,
funny int,
review_id string,
stars int,
text string,
useful int,
user_id string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/review';

CREATE TABLE test.review_filtered
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/review_filtered'
AS
SELECT re.business_id, r.stars, r.user_id
FROM test.review r JOIN test.restaurants re
ON r.business_id = re.business_id;

CREATE EXTERNAL TABLE test.users (
average_stars double,
compliment_cool int,
compliment_cute int,
compliment_funny int,
compliment_hot int,
compliment_list int,
compliment_more int,
compliment_note int,
compliment_photos int,
compliment_plain int,
compliment_profile int,
compliment_writer int,
cool int,
elite array<int>,
fans int,
friends array<string>,
funny int,
name string,
review_count int,
useful int,
user_id string,
yelping_since string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION '/user/training/yelp/users';

CREATE TABLE test.elite_users
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/users/elite'
AS
SELECT * FROM test.users LATERAL VIEW explode(elite) c AS elite_year;

CREATE EXTERNAL TABLE test.tip (
text string,
date_tip string,
likes int,
business_id string,
user_id string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/mboldin/yelp/tip';


```
- validation sqls
```

SELECT * FROM business4 LATERAL VIEW explode(categories) c AS cat_exploded limit 10;

SELECT * FROM exploded LIMIT 1;

SELECT count (DISTINCT business_id) number_businesses FROM exploded;

SELECT count (business_id) number_restaurants FROM exploded
WHERE cat_exploded="Restaurants";

SELECT name, review_count, stars, cat_exploded category FROM restaurants LIMIT 5;

SELECT name, attributes.ambience.romantic FROM restaurants LIMIT 5;

SELECT name, state, city, attributes.ambience.romantic romantic FROM restaurants
WHERE attributes.ambience.romantic = true LIMIT 10;

SELECT * FROM review LIMIT 1;

SELECT count(*) FROM review;

SELECT count(*) FROM review_filtered;


SELECT count(distinct user_id) FROM users;
select count(*) from elite_users;
select count(*) from tip;

```
