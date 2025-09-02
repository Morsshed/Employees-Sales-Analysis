# Employees-Sales-Analysis
This project focuses on analyzing employees’ sales performance using SQL. The goal is to evaluate productivity, identify top performers, and generate insights that can help optimize business decisions.


Question-1: find out the top 5 customers based on amount spent and total orders

                 SELECT 
                	customer_id,
                	ROUND(SUM(AMOUNT)) AS CUSTOMER_PURCHASE 
                 FROM SALES
                	GROUP BY CUSTOMER_ID
                    ORDER BY CUSTOMER_PURCHASE DESC
                    LIMIT 5;

Question-2: find out the top 5 customers based on total orders

                  SELECT 
                  	  S. customer_id,
                      NAME,
                  	  COUNT(DISTINCT SALE_ID) AS TOTAL_ORDERS
                   FROM SALES S 
                  	LEFT JOIN CUSTOMERS C  
                      ON S. CUSTOMER_ID = C. CUSTOMER_ID
                  	GROUP BY S. CUSTOMER_ID, NAME
                      ORDER BY TOTAL_ORDERS DESC
                      LIMIT 5;
   
Question-3: Rank the top 5 customers based on total orders (Solve the problem with Window Function)
   
                SELECT 
                	  S. customer_id,
                    NAME,
                	  COUNT(DISTINCT SALE_ID) AS TOTAL_ORDERS,
                    RANK() OVER (ORDER BY COUNT(DISTINCT SALE_ID) DESC) AS CUSTOMER_RANK
                 FROM SALES S 
                	LEFT JOIN CUSTOMERS C  
                    ON S. CUSTOMER_ID = C. CUSTOMER_ID
                	GROUP BY S. CUSTOMER_ID, NAME
                    ORDER BY TOTAL_ORDERS DESC
                    LIMIT 5;
   
Question-4: Rank the top 5 customers based on total orders (Solve the problem with Sub-Query)
  
                SELECT *
                FROM (
                 SELECT 
              	   S. customer_id,
                   NAME,
              	   COUNT(DISTINCT SALE_ID) AS TOTAL_ORDERS,
                  RANK() OVER (ORDER BY COUNT(DISTINCT SALE_ID) DESC) AS CUSTOMER_RANK
               FROM SALES S 
              	LEFT JOIN CUSTOMERS C  
                  ON S. CUSTOMER_ID = C. CUSTOMER_ID
              	GROUP BY S. CUSTOMER_ID, NAME
                  ORDER BY TOTAL_ORDERS DESC) AS A
                  WHERE CUSTOMER_RANK <= 5 ;
    
Question-4: Rank the top 5 customers based on total orders (Solve the problem with CTE)
    
                   WITH A AS 
                  	(
                  SELECT 
                  	  S. customer_id,
                      NAME,
                  	  COUNT(DISTINCT SALE_ID) AS TOTAL_ORDERS,
                      RANK() OVER (ORDER BY COUNT(DISTINCT SALE_ID) DESC) AS CUSTOMER_RANK
                   FROM SALES S 
                  	LEFT JOIN CUSTOMERS C  
                      ON S. CUSTOMER_ID = C. CUSTOMER_ID
                  	GROUP BY S. CUSTOMER_ID, NAME
                      ORDER BY TOTAL_ORDERS DESC) 
                  SELECT * FROM A
                      WHERE CUSTOMER_RANK <= 5 ; 
    
Question-5: Show the difference between Rank and Dense_Rank based on the top customers on their total orders.
    -- USING CTE FOR MULTI-PURPOSE WITH DENSE RANK ()
                      
                  WITH A AS 
                  	(
                  SELECT 
                  	  S. customer_id,
                      NAME,
                  	  COUNT(DISTINCT SALE_ID) AS TOTAL_ORDERS,
                      RANK() OVER (ORDER BY COUNT(DISTINCT SALE_ID) DESC) AS CUSTOMER_RANK
                   FROM SALES S 
                  	LEFT JOIN CUSTOMERS C  
                      ON S. CUSTOMER_ID = C. CUSTOMER_ID
                  	    GROUP BY S. CUSTOMER_ID, NAME
                        ORDER BY TOTAL_ORDERS DESC) 
                  SELECT 
                  	  CUSTOMER_ID,
                      NAME,
                      TOTAL_ORDERS,
                      CUSTOMER_RANK,
                      DENSE_RANK () OVER (ORDER BY TOTAL_ORDERS DESC) AS DENSE_RANK_OF_CUSTOMER
                  FROM A;

                  -- WHERE CUSTOMER_RANK <= 5 ; 
                    
 
SELECT * FROM EMPLOYEES;
SELECT * FROM CUSTOMERS;
SELECT * FROM SALES;


Question-6: Get the top 10 employees serial whose total sales are higher than average sales 

                              -- STEP-1: AVERAGE SALES AMOUNT-- 

                      SELECT 
                          ROUND(AVG(AMOUNT),2) AS SALES_AMOUNT
                          FROM SALES; 
    
                             -- STEP-2: EMPLOYEES WISE TOTAL SALES--
  
                      SELECT 
                      	  EMP_ID,
                          SUM(AMOUNT) TOTAL_SALES
                          FROM SALES
                      GROUP BY EMP_ID; 

                              -- STEP-3: JOINING THE NAME--

                      SELECT 
                      	  E.EMP_ID,
                          E.NAME,
                          SUM(AMOUNT) TOTAL_SALES
                        FROM SALES S
                        LEFT JOIN EMPLOYEES E 
                          ON S.EMP_ID = E.EMP_ID
                          GROUP BY E.EMP_ID, E.NAME; 

                                  -- STEP-4: from Query 3 AND 1--
                      
                      WITH SALES_CTE AS (
                      SELECT 
                          ROUND(AVG(AMOUNT),2) AS SALES_AMOUNT
                          FROM SALES)
                      
                      SELECT 
                      	E.EMP_ID,
                          E.NAME,
                          SUM(AMOUNT) AS TOTAL_SALES,
                          ROW_NUMBER() OVER(ORDER BY SUM(AMOUNT) DESC) AS POSITION
                          FROM SALES S
                          LEFT JOIN EMPLOYEES E 
                          ON S.EMP_ID = E.EMP_ID
                      GROUP BY E.EMP_ID, E.NAME
                      HAVING SUM(AMOUNT) > (SELECT SALES_AMOUNT FROM SALES_CTE)
                      LIMIT 10; 


Question-7: Which employees have made sales above the average sale amount? (Both CTE and Sub-Query)

                        WITH AVR_SALES_CTE AS (
                        SELECT
                        	AVG(AMOUNT) AS AVR_SALES
                        FROM SALES)
                        
                        SELECT 
                        	  SALES.EMP_ID,
                            NAME,
                            AMOUNT
                        FROM SALES
                        	LEFT JOIN EMPLOYEES
                            ON SALES.EMP_ID = EMPLOYEES.EMP_ID
                        WHERE AMOUNT > (SELECT AVR_SALES FROM AVR_SALES_CTE);

   
Question-8: Find out Which employees average sales is above the average sale amount? (individual average sales VS total average sales)

                        
                        SELECT 
                        	  S.EMP_ID,
                            NAME,
                        	ROUND(AVG(AMOUNT),2) AS INDIVIDUAL_AVG_SALES
                            FROM SALES S
                            LEFT JOIN EMPLOYEES E
                            ON S.EMP_ID = E.EMP_ID
                        GROUP BY S.EMP_ID, NAME
                          HAVING ROUND(AVG(AMOUNT),2) >(SELECT AVG(AMOUNT)FROM SALES);
                          
  
Question-9: find customers who made their first purchase within 7 days of signing up?

                          -- Step-1: First Purchase Date-- 
                        
                        SELECT 
                        	MIN(SALE_DATE) AS FIRST_PURCHASE
                        FROM SALES;

                        -- Step-2: Customers' Purchases with 7 days of signup--
                            
                         select 
                        	  SALES.CUSTOMER_ID,
                            CUSTOMERS.NAME,
                            customers.SIGNUP_DATE,
                            MIN(SALE_DATE) AS FIRST_PURCHASE_DATE,
                            datediff(MIN(sales.SALE_DATE), customers.signup_date) as days
                        FROM SALES
                        	LEFT JOIN CUSTOMERS
                            ON SALES.CUSTOMER_ID = CUSTOMERS.CUSTOMER_ID
                        	GROUP BY sales.CUSTOMER_ID,
                        			 CUSTOMERS.NAME,
                                     SIGNUP_DATE
                                     having datediff(MIN(sales.SALE_DATE), customers.signup_date) <= 7 
                        				and datediff(MIN(sales.SALE_DATE), customers.signup_date) >= 0 ;
    

