# Challenge 22 - Big Data

This challenge demonstrates the usage of PySpark and SparkSQL using home sale data from 2019 to 2022.

The data is loaded from an S3 bucket but can also be found in this repository at [`Resources/home_sales_revised.csv`](Resources/home_sales_revised.csv). The analysis is done in the [Home_Sales.ipynb](Home_Sales.ipynb) notebook.

---

I analyzed the data to answer the following questions:
1. What is the average price for a four-bedroom house sold for each year?
    ```sql
    SELECT
        YEAR(date) AS year,
        ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    WHERE bedrooms=4
    GROUP BY year
    ORDER BY year
    ```
    - 2019: 300,263.70
    - 2020: 298,353.78
    - 2021: 301,819.44
    - 2022: 296,363.88
2. What is the average price of a home for each year it was built that has three bedrooms and three bathrooms?
    ```sql
    SELECT
        YEAR(date) AS year,
        ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    WHERE bedrooms=3 AND bathrooms=3
    GROUP BY year
    ORDER BY year
    ```
    - 2019: 287,287.82
    - 2020: 294,204.16
    - 2021: 294,211.46
    - 2022: 292,725.69
3. What is the average price of a home for each year that has three bedrooms, three bathrooms, two floors, and is greater than or equal to 2,000 square feet?
    ```sql
    SELECT
        YEAR(date) AS year,
        ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    WHERE
        bedrooms=3
        AND bathrooms=3
        AND floors=2
        AND sqft_living>=2000
    GROUP BY year
    ORDER BY year
    ```
    - 2019: 289,859.14
    - 2020: 292,289.09
    - 2021: 296,330.96
    - 2022: 290,242.99
4. What is the "view" rating for the average price of a home where the homes are greater than or equal to $350,000?
    ```sql
    SELECT
        view,
        ROUND(AVG(price), 2) AS avg_price
    FROM home_sales
    GROUP BY view
    HAVING avg_price >= 350000
    ORDER BY view DESC
    ```
    - 50 results are returned

The last question was used to compare run time with a cached table and partitioned parquet data (with the query being modified to use the partitioned table). The partition was by the "date_built" field. The results of running the query five times are shown in the table below in seconds:

|              | 1 (load) | 2       | 3       | 4       | 5       | Avg of 2-5 |
|:------------:|---------:|--------:|--------:|--------:|--------:|-----------:|
| **Standard** | 0.3427   | 0.2151  | 0.2471  | 0.2294  | 0.1914  | 0.2207     |
| **Cached**   | 0.1442   | 0.1483  | 0.1476  | 0.1544  | 0.1376  | 0.1469     |
| **Parquet**  | 0.4035   | 0.3123  | 0.2820  | 0.2579  | 0.2505  | 0.2756     |

The first run was not included in the average as it was when the cell was run for the first time.

### Conclusion
From the time table, we can see that running a query on the cached table was the fastest option. We also see that the partitioned parquet files were the slowest. This is likely due to the partition being on the "date_built" column, which was not used in the query and the size of the data was not very big.
