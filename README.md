# NovaPay
## Fraudulent Transaction detection for digital money transfer

### Data Cleaning
Data types were fixed and timestamp was converted from object to datetime and amount_src was converted from object to float

#### Filling in missing values for targeted columns

Filling in missing values for amount_usd carried out by calculating exchange rates per currency by selecting non-missing rows in amount_usd and grouping by source currency, then computing the mean of amount_usd/amount_src for each currency and saving the results to a dictionary for easy lookup. The missing values in amount_src are filled by multiplying the values derived for the exchange rate and the source_currency column

Filling in missing values for the fee column is done based on the median for the channel feature since fee is dependent on channel

Filling in the ip_country values done by using the corresponding home country

Filling in the missing values in kyc_tier with the mode

Filling in the missing values in device_trust_score using the group's median value

#### Droppig missing values and duplicated rows
The rows that had missing values for ip_address(305), timestamp(61) and amount_src(4) were dropped as they are unique values that cannot be computed
Duplicated rows which totaled to 194 rows were also dropped

### Sanity Checks for the dataset

This section lists essential sanity checks to validate the dataset after cleaning and imputation

1. Check for invalid numeric values, including negative values in monetary, risk, trust or velocity fields and validate the user age in days is not negative
2. Verify currency-related logic and ensure there are no negative values and that the derived exchange rates fall within reasonable range
3. Validate timestamp integrity and confirm no transaction timestamps occur in the future
4. Review location consistency to ensure the counts in location_mismatch was generated corrrectly and the country features contain plausible country codes
5. Check categorical column consistency to review unique values in channel, source_currency, dest_currency and kyc_tier to ensure there are no unexpected entries
6. Validate risk-score ranges ensuring all values fall within expected numeric range
7. Ensure standardization of formatting used in the features
8. Confirm fraud label integrity ensuring they contain only binary values
9. Validate velocity features

These checks ensure this dataset is thoroughly consistent before the feature engineering and modeling stage.
