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

The rows that had missing values for ip_address(420), timestamp(61) and amount_src(4) were dropped as they are unique values that cannot be computed
