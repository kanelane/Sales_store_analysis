---select * from sales_store

--checking for duplicates

select transaction_id, count(*)
from sales_store
group by transaction_id
having COUNT(transaction_id) >1

with CTE as (
select *,
    ROW_NUMBER() over (partition by transaction_id order by transaction_id) as row_num
	from sales_store
	)
	--delete from CTE
	--where row_num=2

	select* from cte
	where row_num >=1

	--step 1; completed successfully deleted all duplicates

	--step 2; cerrection of headers

	EXEC sp_rename 'sales_store.quantiy','quantity' ,'COLUMN'

	EXEC sp_rename 'sales_store.prce','price' ,'COLUMN'

	select * from sales_store

	--checking for datatype
	--select column_name, data_typr
	--from infromation_schema.columns
	--where table_name='sales_store'


	--step 4: checking for null values

	select * from sales_store
	where transaction_id is null
	or
	customer_id is null
	or
	customer_name is null
	or
	customer_age is null
	or
	gender is null
	or
	product_id is null
	or
	product_name is null
	or
	product_category is null
	or
	quantity is null
	or
	price is null
	or
	payment_mode is null
	or
	purchase_date is null
	or
	time_of_purchase is null
	or status  is null
	
	delete from sales_store
	where transaction_id is null

	select * from sales_store
	where customer_name ='Ehsaan Ram'

	---That shows ehsaaan Ram made multiples purchases and therefore he has the customer_id and so we will replace it.

	update sales_store
	set customer_id='CUST9494'
	where transaction_id='TXN977900'

	--Now we check for Damini Raju

	select * from sales_store
	where customer_name= 'Damini Raju'

    ---That also apears that damini raju also made multiple purchases and is a regular

	update sales_store
	set customer_id ='CUST1401'
	where transaction_id= 'TXN985663'

	---Now both our guys are sorted...
	---Now we sort the cutomer_id CUST1003

	select* from sales_store
	where customer_id='CUST1003'

	update sales_store
	set customer_name='Mahika Saini', customer_age=35, gender='Male'
	where customer_id='CUST1003'

	--Done fixing nulls

	select* from sales_store

	---Step 5: Data cleaning

	select distinct gender
	from sales_store
	--gender looks to be incosistent

	update sales_store
	set gender ='M'
	where gender = 'Male'

    update sales_store
	set gender ='F'
	where gender = 'Female'

	--gender done

	--now we fixing the payment mode 

	select distinct payment_mode
	from sales_store

	update sales_store
	set payment_mode= 'Credit Card'
	where payment_mode= 'CC'

	---Done fixing payment mode and Data cleaning

	--step 5: Solving Business Insights Questions

	--Data analysis-

	-- 1. What are the top 5 most selling products by quantity?

	select top 5 product_name, SUM(quantity) as total_quantity_sold
	from sales_store
	where status = 'delivered'
	group by product_name
	order by total_quantity_sold desc
   
   --status column matters beacause some items are either pending, returned, or cancelled.
   select distinct status
   from sales_store

   -- Wardrobe and Vegetables looks to be the one in demand.
   --Done for the first question--

   --Business problem: we did no know which product are mostly in demand.

   --Business impact: helps prioratize stock and boost sales through targeted promotions

   --2. Which products  are most frequently cancelled?

   select top 5 product_name, count(*) as total_cancelled
   from sales_store
   where status='cancelled'
   group by product_name
   order by total_cancelled desc

   --Business problem: frequent cancellations affect revenue and customer trust. 
   ---Comics and Sweater are the most cancelled products.

   --Business Impact: Identify poor_products to improve quality or remove from catalog...

   --3. What time of the day has the highest number of purchases?

   select * from sales_store

    select
	    case
		     when DATEPART(HOUR, time_of_purchase) between 0 and 5 then 'Night'
			 when DATEPART(HOUR, time_of_purchase) between 6 and 11 then 'Morning'
			 when DATEPART(HOUR, time_of_purchase) between 12 and 17 then 'Afternoon'
			 when DATEPART(HOUR, time_of_purchase) between 18 and 23 then 'Evening'
		end as time_of_day,
		count(*) as total_orders
		from sales_store
		group by
		  case
		     when DATEPART(HOUR, time_of_purchase) between 0 and 5 then 'Night'
			 when DATEPART(HOUR, time_of_purchase) between 6 and 11 then 'Morning'
			 when DATEPART(HOUR, time_of_purchase) between 12 and 17 then 'Afternoon'
			 when DATEPART(HOUR, time_of_purchase) between 18 and 23 then 'Evening'
		end
		order by total_orders desc

		--Business problem solved : find peak times, which is morning and evening
		--Busimess impact: Optimize staffing, promotions, and server loads.

		--4. who are the top 5highest spending customers

		select top 5 customer_name, SUM(price*quantity) as total_spend
		from sales_store
		group by customer_name
		order by total_spend desc
		  
		--business problem solved: identify VIP customers
		--business impact: personalised offers, loyalty rewards and retention.


		--5.which product category generate the highest revenue?

		select product_category,sum(price*quantity) as revenue
		from sales_store
		group by product_category
		order by revenue desc

		--Accessories

		--business problem solved: identify top-performing product categories

		--business impact: refine product strategy, supply chain, and promotions,
		--allowing the business to invest more in high-margin or high-demand categories.


		--6.what is the return/cancellation rate per product category?

		--cancellation
		select product_category,
		count(case when status='cancelled' then 1 end)*100.0/count(*) as cancelled_percent
		from sales_store
		group by product_category
		order by cancelled_percent desc

		--books
		--Return
		select product_category,
		count(case when status='returned' then 1 end)*100.0/count(*) as returned_percent
		from sales_store
		group by product_category
		order by returned_percent desc

		--Accessories
		
		--Business problem solved: Monitor dissatisfaction trends per category.

		--Business impact: Reduce returns, improve product descriptions/ expectations.
		--Helps identify and fix product or issues.

		--7. What is the most freferred payment mode?


		select payment_mode,COUNT(payment_mode) as total_count
		from sales_store
		group by payment_mode
		order by total_count desc

		--They like credit card

		--business problem solved: know which payment options customers prefer.

		--business impact: streamline payment processing, prioritize popular modes.

		--8.How does age group afffect purchasing behavior?
		
		select
		      case
			      when customer_age between 18 and 25 then '18-25'
				  when customer_age between 26 and 35 then '26-35'
				  when customer_age between 36 and 50 then '36-50'
				  else '51+'
			  end as customer_age,
			  sum(price*quantity) as total_purchase
			  from sales_store

			group by case
			  when customer_age between 18 and 25 then '18-25'
				  when customer_age between 26 and 35 then '26-35'
				  when customer_age between 36 and 50 then '36-50'
				  else '51+'

			end
			order by total_purchase desc

			--business prblem solved: understand customer demographics.
			--36-50
			--business impact: Targeted marketing and product recommendations by age group.

			--9.whats the montlhy sales trend?

			--select
			--year(purchase_date) as years,
			--month(purchase_date) as months,
			--SUM(quantity) as total_quantity
			--from sales_store
			--group by year(purchase_date),month(purchase_date)
			--order by months
			
			--10.Are certain genders buying more specific product categories?

			---method 1

			select gender, product_category,count(product_category) as total_purchase
			from sales_store
			group by gender, product_category
			order by gender

			--Business problem solved: gender-based product preferences.

			--Business impact: Personalised ads, gender-focused campaigns.