# Project Documentation 

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
 * **tax_rate classiification** - Grouping of tax rate for country into high and low.
 * **young_selfmade** - Billionaires that are young based on the threshold defined and are selfmade.

## Data Processing 
1. **Data Cleaning**
   A. Inconsistency: Wrong spelling was checked for in the names of billionaires and corrected to the right value
   
   /*Checking and Correcting Errors in Names*/
      ```sql
      -- Select records with first names starting with "Fran"
      SELECT * 
      FROM demographics_info
      WHERE firstName LIKE "Fran%";
      
      -- Update records to correct the name
      UPDATE demographics_info
      SET firstName = "Francois"
      WHERE firstName = "FranÃƒÂ§ois";





