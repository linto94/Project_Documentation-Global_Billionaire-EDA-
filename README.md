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
   4. 

   





