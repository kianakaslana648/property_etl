# Property ETL

### Goals

Write code (python preferred) to integrate the two datasets, considering data quality, consistency, and scalable ETL process design.
Convert the attached JSON files into relational tables in an Excel file. Key tables should include property locations, descriptions, tax history, etc.

Identify data issues and give solutions to fix them.

Integrate the two datasets and remove duplicate information. Note that there can be similar columns with different names, for example, Bath and Bathroom.

Compare your output data with the attached schema template and identify missing attributes.

### Architecture

Since startups always place a focus on the iterative ability on the architecture, ***snowflake*** schema is perferred to achieve both iterative ability and logical isolation (expect to expand in future following the Kimball Methodology)

For this case study, I chose to construct a fact table for tax history assessment ***factTaxHistory***, a main dimensional table for properties ***dimProperties***, and subdimensional tables for addresses, descriptions and utilities ***dimAddress***, ***dimDescription***, ***dimUtility***.

I utilized ***Spark*** to implement the ETL processes to extract and integrate the source data of scraped json files from ***Zillow*** and ***Realtor***, transform the data into a relational table schema, load into the destination of an excel file. (In practical cases, insert or upsert actions should be operated to merge data into a data warehouse)

Quick access to the report page: [Report.html](https://kianakaslana648.github.io/property_etl/)

### File Use
- Property_ETL.ipynb, the notebook doing the whole ETL process;
- Report.html, the report explaining the approaches used;
- Report.ipynb, the notebook used to generate the report;
- requirement.txt, the python dependencies required;
- result_table.xlsx, the result excel file containing the result tables;
- ZIP_COUNTY_122023.xlsx, the file used in Property_ETL.ipynb to convert zip code into county fips code.

### Data Consistency

To keep data consistency among different data sources, there are approaches to set feature vectors for each record and common ML methods (such as GlueETL FindMatches) to deduplicate. However, in this case, the pair of latitude and longitude could be used to identify a unique property. We can prove that through merging by or deduplicating by the (lat, lon) pair of rounded-up values to 3 decimals, we can set a control that property records with distance detected within 80 metres are recognized as the same property. (Or more strictly, choosing to round up to 4 decimals leads to a limit of 8 metres)

**Haversine Formula**

We can use latitude and longitude to identify the properties by using (lat, lon) pair.

We can prove that when we have a maximum bias of $10^{-4}$ for both lat and lon, we will control the distance of two places under 10 meters; and when we have a maximum bias of $10^{-3}$ for both lat and lon, we will control the distance of two places under 80 meters. So, by rounding up the latitude and longitude to four decimal places, we can deduplicate by (lat, lon) pair to ensure two properties are at least in the same building. So we can deduplicate by (lat, lon) pair plus some other attributes of properties.

See more details of the proof in **Report.html**

### Some strategies to handle the Data Quality Issues

- Address features such as county and FIPS code are not ensured to exist.
    - Use files from [www.huduser.gov](https://www.huduser.gov/portal/datasets/usps_crosswalk.html) to infer the missing values from existing address features.
- In the Realtor dataset, property features such as area, price, bedrooms, bathrooms are not ensured to exist when the struct feature of units exist.
    - Use sum of living area, listing price, bedroom number, bathroom number for the units as the feature of the property.
- When identifying the same property, some features are recorded differently.
    - Use features extracted from Zillow prioritized over features from Realtor.
- Tax histories are not ensured to be recorded for each property.
    - By building a fact table for tax histories, we focus on existing tax histories.
- Boolean values such as flags indicating the existence of a garage, a pool, a cooling system or a heating system are very sparse.
    - Null values are transformed into False values.
- Area features such as living area and lot size could be in different units.
    - Store the features in string types with the unit concatenated.
- For categorical types, there are values representing the same type but with minor difference in different data sources.
    - While edit distance could be useful in this case, a formal way is to create a standardized dictionary and transform original values.
 
### Insights into the ATTOM Schema

As a standard schema for the properties, the ATTOM Schema has evolved for tens of years and has been elaborately designed for storing detailed information in different groups of topics. Some points can be learned from the perspective of a startup to build a data warehouse:
- Abundant domain knowledge of properties could be learned from the ATTOM schema. From the perspective of business intelligence, valuable property information includes legal information, party owners, contact owners, deed owners, etc.
- While the ATTOM schema keeps heavy data redundancy for the convenience of business analyses (such as splitting street address into several columns, keeping isolated sections for information of attic, parking, garage), it somehow tends to guide the future trends of property data schema. However, from the view of a startup company, heavy data redundancy could lead to low data quality (such as data sparsity) and a burden of infrastructure cost (compute and storage).
- The ATTOM schema sets strict data types for each column. But startups focus more on the iterative ability of the architecture. In the early phase of development, it should be avoided that excessive effort is put into optimization.

