# LandTech--Suresh-Regmi

# Q1 Testing

**Understanding of the problem**

The test provided checks the differences/discrepancies in the number of active contracts between the raw data and the data processed in the contracts dimension table (probably in the warehouse).

Using the common table expression, the **active_raw_contracts** table is created by selecting contracts from salesforce tables where the contract start date is less than or equal to the current date the current date is less than or equal to the coalesced date cancelled and the contract end date. This means the active_Raw_contracts table contains all contracts that are active regardless of whether they have been processed in the data warehouse.

The **active_processed_contracts** on the other hand is created by selecting all contracts from the dim_contracts dimension where the source is salesforce the start date is less than or equal to the current date and the current date is less than or equal to the end date. This means the active_processed_contracts table contains all contracts that are currently active and have been processed in the data warehouse.

The test overall compares the number of active contracts in the two tables. 

**Solution**

If the test is failing, then it means that there is a discrepancy in the number of contracts between the raw data and processed data. This could be due to a number of factors such as:

1. Issue in the data processing logic
2. Difference in the definition of active contract between the raw and processed data source
3. Mismatch or data between two systems


If the issue is being caused by the mismatch in the data between two systems and we need to find the accounts causing this issue, we can use the following steps to find a solution

1. Merging two tables (active_raw_contracts and active_processed_contracts based on the account_id column)
2. Calculating the difference in ARR between two tables
3. Filtering the merge records based on the ARR field. The records which have a difference not equal to zero is the account that is causing the problem.


Here is a sample SQL script to get the solution
```
SELECT arc.account_id as account_id, arc.account_name as account_name, arc.arr as raw_arr, apc.arr as processed_arr, arc.arr-apc.arr as arr_diff
FROM active_raw_contracts arc 
JOIN active_processed_contracts apc 
ON arc.account_id=apc.account_id
WHERE arc.arr != apc.arr
ORDER BY arr_diff DESC;
```
