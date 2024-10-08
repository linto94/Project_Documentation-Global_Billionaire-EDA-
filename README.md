# Project Documentation 📁📝

## Data Preparation
The dataset contains detailed information on the world's billionaires, focusing on their wealth, demographics, industry involvement, and country-specific economic factors. It includes the following key attributes:
 * **finalWorth**: Net worth of the individual in millions.
 * **lastName, age, firstName, birthYear, birthMonth, birthDay, gender**: Information about the individual's identity and age.
 * **self-made**: Whether the billionaire is self-made or inherited their wealth (True/False).
 * **Industries**: The primary industry in which the billionaire operates (e.g., technology, finance).
 * **source**: The source of wealth, often listing companies they are associated with.
 * **country, city**: The country and city where the individual is based.
 * **cpi_country**: Consumer Price Index of the individual's country.
 * **gdp_country**: Gross Domestic Product of the country.
 * **life_expectancy_country**: Average life expectancy in the country.
 * **tax_revenue_country, total_tax_rate_country**: Tax revenue and total tax rate in the billionaire's country.
 * **population_country**: Population of the country.
 * **tax_rate classiification** - Classification of tax rate for country into high and low.
 * **young_selfmade** - Billionaires that are young based on the threshold defined and are selfmade.

## Data Processing 
1. **Data Cleaning**
   1. **Inconsistency**: Wrong spelling was checked for in the names of billionaires and corrected to the right value
      ```sql
      -- Select records with first names starting with "Fran"
      SELECT * 
      FROM demographics_info
      WHERE firstName LIKE "Fran%";
      
      -- Update records to correct the name
      UPDATE demographics_info
      SET firstName = "Francois"
      WHERE firstName = "FranÃƒÂ§ois";
      
   2. **Inaccuracy**: Outliers were detected and corrected in the age and birthyear variables as most individuals had negative age which is wrong, coming from their birthyear being in a future year that has not reached, this was detected using skewness and kurtosis with Excel and Zscore in SQL. To confirm the outilers existence a boxplot and historgarm were also created to show the distrubtion of this variables. 
      ```sql
      -- Calculate the Z-score to detect outliers
      SELECT age,
             birthYear,
             lastName,
             (age - AVG(age) OVER()) / STDDEV(age) OVER() AS Zscore
      FROM demographics_info;
      
      -- Using the Z-score to find values in age and birth year outside (above/below) 2 standard deviations from the mean
      SELECT * 
      FROM (
          SELECT age,
                 birthYear,
                 lastName,
                 (age - AVG(age) OVER()) / STDDEV(age) OVER() AS Zscore
          FROM demographics_info
      ) score_table
      WHERE Zscore > 1.96 OR Zscore < -1.96;

      -- Remove outliers from the age and birth year columns
      UPDATE demographics_info
      SET age = 95,
          birthYear = 1929
      WHERE age = -5 AND birthYear = 2029;
      
      UPDATE demographics_info
      SET age = 96,
          birthYear = 1928
      WHERE age = -4 AND birthYear = 2028;
      
      -- Verify results
      SELECT *
      FROM demographics_info
      WHERE age = -5 OR age = -4;
      
   ![18](https://github.com/user-attachments/assets/55d04d3a-6386-4237-8079-38fc56d2fc4c)
   
   ![18a](https://github.com/user-attachments/assets/82900b27-4a82-4877-b75f-9fcf9797e8ab)
   
   ![19](https://github.com/user-attachments/assets/6480189e-24cf-450c-aebe-05365f13d1ae)
   
   ![19a](https://github.com/user-attachments/assets/cdabeec2-4ef3-4551-a22c-a7e7c961e758)


2. **Data Transformation**
   1. **Remove unwanted columns not needed for the analysis**
      ```sql
      -- Delete the category variable from the table as it is the same as industries
      ALTER TABLE demographics_info
      DROP COLUMN category;

   2. **Spilt a Column into two** - The name column was separated into two columns (last and first name)
      ```sql
      -- Add new columns for last name and first name
      ALTER TABLE demographics_info 
        ADD lastName VARCHAR(255),
        ADD firstName VARCHAR(255);
      
      -- Update the values in the new columns
      UPDATE demographics_info 
      SET 
          lastName = SUBSTRING_INDEX(name, ';', 1),
          firstName = SUBSTRING_INDEX(name, ';', -1);
      
      -- Delete the name column from the table 
      ALTER TABLE demographics_info 
      DROP COLUMN name;
      
      -- Verify results
      SELECT * FROM demographics_info;
   3. **Standardization** - Standardized final worth column changing the data type to the appropriate one of currency rather than just as a number and expressing it in full (billions) rather than in thousands.  
      ```sql
      -- Make the finalWorth column in full billions rather than thousands
      UPDATE demographics_info
      SET finalWorth = finalWorth * 1000 * 1000;
      
   4. **Feature Engineering** - Extracted new variables (columns) from birthdate such as birthYear, birthMonth, birthDay and calculated the Age using the current year – birthyear, also extracted new variables which includes young_selfmade and tax_rate_classfication using conditional IFELSE statement for the aim of exploration.
      ```sql
      -- Add new columns for birth year, birth month, birth day, and age
      ALTER TABLE demographics_info
        ADD birthYear INT,
        ADD birthMonth VARCHAR(255),
        ADD birthDay INT,
        ADD age INT;
      
      -- Populate the new columns with values
      UPDATE demographics_info
      SET 
         birthYear = YEAR(birthDate),
         birthDay = DAY(birthDate),
         age = 2024 - birthYear;
      
      UPDATE demographics_info
      SET 
         birthMonth = CASE 
             WHEN MONTH(birthDate) = 1 THEN 'January'
             WHEN MONTH(birthDate) = 2 THEN 'February'
             WHEN MONTH(birthDate) = 3 THEN 'March'
             WHEN MONTH(birthDate) = 4 THEN 'April'
             WHEN MONTH(birthDate) = 5 THEN 'May'
             WHEN MONTH(birthDate) = 6 THEN 'June'
             WHEN MONTH(birthDate) = 7 THEN 'July'
             WHEN MONTH(birthDate) = 8 THEN 'August'
             WHEN MONTH(birthDate) = 9 THEN 'September'
             WHEN MONTH(birthDate) = 10 THEN 'October'
             WHEN MONTH(birthDate) = 11 THEN 'November'
             WHEN MONTH(birthDate) = 12 THEN 'December'
             ELSE 'Unknown'
         END;

         -- Add a new column to classify young self-made billionaires
         ALTER TABLE billionaire_data
         ADD young_selfmade VARCHAR(255);
         
         UPDATE billionaire_data
         SET 
             young_selfmade = CASE
                 WHEN selfMade = 'True' AND age <= 50 THEN 'Yes'
                 ELSE 'No' 
             END;
         
         -- Add a new column to classify tax rates into High or Low
         ALTER TABLE billionaire_data
         ADD taxrate_classification VARCHAR(255);
         
         UPDATE billionaire_data
         SET 
             taxrate_classification = CASE 
                 WHEN total_tax_rate_country >= 43 THEN 'High' 
                 ELSE 'Low' 
             END;


   5. **Join** - Performed Inner Joint to combine the demographics_data with the economic_indcator_data  to a single dataset.
         ```sql
         -- Combine demographic data with economic indicators data
         SELECT *
         FROM billionaire.demographics_info
         JOIN billionaire.economic_indicators
         ON demographics_info.country = economic_indicators.country;
         
         -- Remove duplicates due to the join operation
         CREATE TABLE billionaire_data AS
         SELECT DISTINCT
             di.country,
             di.finalWorth,
             di.city,
             di.source,
             di.industries,
             di.selfMade,
             di.gender,
             di.birthDate,
             di.lastName,
             di.firstName,
             di.birthYear,
             di.birthMonth,
             di.birthDay,
             di.age,
             ei.cpi_country,
             ei.gdp_country,
             ei.life_expectancy_country,
             ei.tax_revenue_country_country,
             ei.total_tax_rate_country,
             ei.population_country
         FROM billionaire.demographics_info di
         JOIN billionaire.economic_indicators ei
         ON di.country = ei.country;

## Guiding Questions (Analysis)
1. **Wealth and Industry**
    * Which industries dominate among billionaires?
         ```sql
         -- Count the number of billionaires in each industry
         SELECT industries, COUNT(*) AS Billionaire_Count
         FROM billionaire_data
         GROUP BY industries
         ORDER BY Billionaire_Count DESC;
         
    * How does the total wealth of billionaires differ across industries and regions?
         ```sql
         -- Calculate total wealth of billionaires by industry
         SELECT industries, SUM(finalWorth) AS Billionaire_Wealth
         FROM billionaire_data
         GROUP BY industries
         ORDER BY Billionaire_Wealth DESC;
         
         -- Calculate total wealth of billionaires by country, limited to the top 10
         SELECT country, SUM(finalWorth) AS Billionaire_Wealth
         FROM billionaire_data
         GROUP BY country
         ORDER BY Billionaire_Wealth DESC
         LIMIT 10;

    * Are there emerging industries creating new billionaires, and what is their impact on wealth distribution?




