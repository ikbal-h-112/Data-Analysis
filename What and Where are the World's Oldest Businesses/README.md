## 1. The oldest business in the world
<p><img src="https://assets.datacamp.com/production/repositories/5851/datasets/7dded924c6dc418d4a828f2f4daba99953c27a5a/400px-Eingang_zum_St._Peter_Stiftskeller.jpg" alt="The entrance to St. Peter Stiftskeller, a restaurant in Saltzburg, Austria. The sign over the entrance shows &quot;803&quot; - the year the business opened."></p>
<p><em>Image: St. Peter Stiftskeller, founded 803. Credit: <a href="https://commons.wikimedia.org/wiki/File:Eingang_zum_St._Peter_Stiftskeller.jpg">Pakeha</a>.</em></p>
<p>An important part of business is planning for the future and ensuring that the company survives changing market conditions. Some businesses do this really well and last for hundreds of years.</p>
<p>BusinessFinancing.co.uk <a href="https://businessfinancing.co.uk/the-oldest-company-in-almost-every-country">researched</a> the oldest company that is still in business in (almost) every country and compiled the results into a dataset. In this project, you'll explore that dataset to see what they found.</p>
<p>The database contains three tables.</p>
<h3 id="categories"><code>categories</code></h3>
<table>
<thead>
<tr>
<th style="text-align:left;">column</th>
<th>type</th>
<th>meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;"><code>category_code</code></td>
<td>varchar</td>
<td>Code for the category of the business.</td>
</tr>
<tr>
<td style="text-align:left;"><code>category</code></td>
<td>varchar</td>
<td>Description of the business category.</td>
</tr>
</tbody>
</table>
<h3 id="countries"><code>countries</code></h3>
<table>
<thead>
<tr>
<th style="text-align:left;">column</th>
<th>type</th>
<th>meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;"><code>country_code</code></td>
<td>varchar</td>
<td>ISO 3166-1 3-letter country code.</td>
</tr>
<tr>
<td style="text-align:left;"><code>country</code></td>
<td>varchar</td>
<td>Name of the country.</td>
</tr>
<tr>
<td style="text-align:left;"><code>continent</code></td>
<td>varchar</td>
<td>Name of the continent that the country exists in.</td>
</tr>
</tbody>
</table>
<h3 id="businesses"><code>businesses</code></h3>
<table>
<thead>
<tr>
<th style="text-align:left;">column</th>
<th>type</th>
<th>meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;"><code>business</code></td>
<td>varchar</td>
<td>Name of the business.</td>
</tr>
<tr>
<td style="text-align:left;"><code>year_founded</code></td>
<td>int</td>
<td>Year the business was founded.</td>
</tr>
<tr>
<td style="text-align:left;"><code>category_code</code></td>
<td>varchar</td>
<td>Code for the category of the business.</td>
</tr>
<tr>
<td style="text-align:left;"><code>country_code</code></td>
<td>char</td>
<td>ISO 3166-1 3-letter country code.</td>
</tr>
</tbody>
</table>
<p>Let's begin by looking at the range of the founding years throughout the world.</p>
%%sql 
postgresql:///oldestbusinesses



SELECT MIN(year_founded), MAX(year_founded)
    FROM businesses;

## 2. How many businesses were founded before 1000?
<p>Wow! That's a lot of variation between countries. In one country, the oldest business was only founded in 1999. By contrast, the oldest business in the world was founded back in 578. That's pretty incredible that a business has survived for more than a millennium.</p>
<p>I wonder how many other businesses there are like that.</p>
%%sql


Select count(*)
From businesses
where year_founded < 1000
## 3. Which businesses were founded before 1000?
<p>Having a count is all very well, but I'd like more detail. Which businesses have been around for more than a millennium?</p>
%%sql

Select *
From businesses
Where year_founded < 1000
Order by year_founded ;
## 4. Exploring the categories
<p>Now we know that the oldest, continuously operating company in the world is called Kongō Gumi. But was does that company do? The category codes in the <code>businesses</code> table aren't very helpful: the descriptions of the categories are stored in the <code>categories</code> table.</p>
<p>This is a common problem: for data storage, it's better to keep different types of data in different tables, but for analysis, we want all the data in one place. To solve this, you'll have to join the two tables together.</p>
%%sql



Select business, year_founded, country_code,category
From businesses as b
Join categories as c
Using (category_code)
Where year_founded <1000
Order by year_founded ;

## 5. Counting the categories
<p>With that extra detail about the oldest businesses, we can see that Kongō Gumi is a construction company. In that list of six businesses, we also see a café, a winery, and a bar. The two companies recorded as "Manufacturing and Production" are both mints. That is, they produce currency.</p>
<p>I'm curious as to what other industries constitute the oldest companies around the world, and which industries are most common.</p>
%%sql


Select category , count(category) as "n"
from categories as c
join businesses as b
Using(category_code)
group by category
order by n desc
limit 10;
## 6. Oldest business by continent
<p>It looks like "Banking &amp; Finance" is the most popular category. Maybe that's where we should aim if you want to start a thousand-year business.</p>
<p>One thing we haven't looked at yet is where in the world these really old businesses are. To answer these questions, we'll need to join the <code>businesses</code> table to the <code>countries</code> table. Let's start by asking how old the oldest business is on each continent.</p>
%%sql



Select MIN(year_founded) as oldest, continent 
from businesses as b
join countries as c
using(country_code)
group by year_founded,continent
order by oldest;
## 7. Joining everything for further analysis
<p>Interesting. There's a jump in time from the older businesses in Asia and Europe to the 16th Century oldest businesses in North and South America, then to the 18th and 19th Century oldest businesses in Africa and Oceania. </p>
<p>As mentioned earlier, when analyzing data it's often really helpful to have all the tables we want access to joined together into a single set of results that can be analyzed further. Here, that means we need to join all three tables.</p>
%%sql


Select business, year_founded, category, country,continent
From businesses as b
Join categories as ca
using(category_code)
Join countries as cn 
using(country_code)
## 8. Counting categories by continent
<p>Having <code>businesses</code> joined to <code>categories</code> and <code>countries</code> together means we can ask questions about both these things together. For example, which are the most common categories for the oldest businesses on each continent?</p>
%%sql



Select continent,category, count(*) as n 
From businesses as b
Join categories as ca
using(category_code)
Join countries as cn
using(country_code)
group by continent,category
## 9. Filtering counts by continent and category
<p>Combining continent and business category led to a lot of results. It's difficult to see what is important. To trim this down to a manageable size, let's restrict the results to only continent/category pairs with a high count.</p>
%%sql



Select continent,category, count(*) as n 
From businesses as b
Join categories as ca
using(category_code)
Join countries as cn
using(country_code)
group by continent,category
having count(*) > 5
order by n desc
