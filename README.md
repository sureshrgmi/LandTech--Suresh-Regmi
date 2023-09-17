# LandTech--Suresh-Regmi

# Q1 Testing

The test provided checks the differences/discrepancies in the number of active contracts between the raw data and the data processed in the contracts dimension table (probably in the warehouse).

Using the common table expression, the active_Raw_contracts table is created by selecting contracts from salesforce tables where the contract start date is less than or equal to the current date the current date is less than or equal to the coalesced date cancelled and the contract end date. This means the active_Raw_contracts table contains all contracts that are active regardless of whether they have been processed in the data warehouse.

The active_processed_contracts on the other hand is created by selecting all contracts from the dim_contracts dimension where the source is salesforce the start date is less than or equal to the current date and the current date is less than or equal to the end date. This means, the active_processed_contracts table contains all contracts that are currently active and have been processed in the data warehouse.

The test in overall compares the number or active contracts in the two tables. 
