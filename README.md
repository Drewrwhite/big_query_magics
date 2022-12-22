# IPython Magics for BigQuery

 _To use these magics, you must first register them. Run the code cell below:_
```
%load_ext google.cloud.bigquery
```
_If, for whatever reason, you need to reload the registration. Run the code cell below:_
```
%reload_ext google.cloud.bigquery
```
_To unload:_
```
%unload_ext google.cloud.bigquery
```
_Running a query from Big Query public data:_
```
%%bigquery
SELECT name, SUM(number) as count
FROM `bigquery-public-data.usa_names.usa_1910_current`
GROUP BY name
ORDER BY count DESC
LIMIT 3
```
_Running a parameterized query:_
```
%%bigquery --params {"corpus_name": "hamlet", "limit": 10}
SELECT word, SUM(word_count) as count
FROM `bigquery-public-data.samples.shakespeare`
WHERE corpus = @corpus_name
GROUP BY word
ORDER BY count DESC
LIMIT @limit
```
## API Reference

IPython Magics

### %%bigquery
IPython cell magic to run a query and display the result as a DataFrame
```
%%bigquery [<destination_var>] [--project <project>] [--use_legacy_sql]
           [--verbose] [--params <params>]
<query>
```
Parameters:
- `<destination_var>`(Optional[line argument]):
<br>
variable to store the query results. The results are not displayed if this parameter is used. If an error occurs during the query execution, the corresponding QueryJob instance (if available) is stored in the variable instead.

- `--destination_table` (Optional[line argument]):
<br>
A dataset and table to store the query results. If table does not exists, it will be created. If table already exists, its data will be overwritten. Variable should be in a format <dataset_id>.<table_id>.

- `--no_query_cache`(Optional[line argument]):
<br>
Do not use cached query results.

- `--project <project>`(Optional[line argument]):
<br>
Project to use for running the query. Defaults to the context project.

- `--use_bqstorage_api`(Optional[line argument]):
<br>
[Deprecated] Not used anymore, as BigQuery Storage API is used by default.

- `--use_rest_api`(Optional[line argument]):
<br>
Use the BigQuery REST API instead of the Storage API.

- `--use_legacy_sql`(Optional[line argument]):
<br>
Runs the query using Legacy SQL syntax. Defaults to Standard SQL if this argument not used.

- `--verbose`(Optional[line argument]):
<br>
If this flag is used, information including the query job ID and the amount of time for the query to complete will not be cleared after the query is finished. By default, this information will be displayed but will be cleared after the query is finished.

- `--params <params>`(Optional[line argument]):
<br>
If present, the argument following the --params flag must be either:

str - A JSON string representation of a dictionary in the format {"param_name": "param_value"} (ex. {"num": 17}). Use of the parameter in the query should be indicated with @param_name. See In[5] in the Examples section below.

dict reference - A reference to a dict in the format {"param_name": "param_value"}, where the value types must be JSON serializable. The variable reference is indicated by a $ before the variable name (ex. $my_dict_var). See In[6] and In[7] in the Examples section below.

- `<query>`(required, cell argument):
SQL query to run. If the query does not contain any whitespace (aside from leading and trailing whitespace), it is assumed to represent a fully-qualified table ID, and the latter’s data will be fetched.

Returns:
A pandas.DataFrame with the query results.
_The google.cloud.bigquery library also includes a magic command which runs a query and either displays the result or saves it to a variable as a DataFrame._
```
%%bigquery --<project><yourprojectid>
# replace "<project><yourprojectid>"" with your own
SELECT 
  COUNT(*) as total_rows
FROM `bigquery-public-data.samples.gsod`
```
# Save output in a variable `df`
```
%%bigquery df --<project><yourprojectid>
SELECT 
  COUNT(*) as total_rows
FROM `bigquery-public-data.samples.gsod`
```
_Using magics to plot data._
```
%%bigquery total_births
SELECT
    source_year AS year,
    COUNT(is_male) AS birth_count
FROM `bigquery-public-data.samples.natality`
GROUP BY year
ORDER BY year DESC
LIMIT 15
```
_Execute following command to call Matplotlib_
```
%matplotlib inline
```
_Plot `total_births` data in Bar Chart using Pandas `DataFrame.plot()` method._
```
total_births.plot(kind="bar", x="year", y="birth_count")
```
_You can further analyze the sample data to retrieve the number of births by weekdays using magics_
```
%%bigquery births_by_weekday
SELECT
    wday,
    SUM(CASE WHEN is_male THEN 1 ELSE 0 END) AS male_births,
    SUM(CASE WHEN is_male THEN 0 ELSE 1 END) AS female_births
FROM `bigquery-public-data.samples.natality`
WHERE wday IS NOT NULL
GROUP BY wday
ORDER BY wday ASC
```
```
births_by_weekday.plot(x="wday")
```