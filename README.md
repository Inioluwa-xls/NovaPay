# NovaPay
Fraudulent Transaction detection for digital money transfer

Data Cleaning
Data types were fixed and timestamp was converted from object to datetime and amount_src was converted from object to float

Filling in missing values for targeted columns

Filling in missing values for amount_usd. This calculates exchange rates per currency by selecting non-missing rows in amount_usd and grouping by source currency, then computing the mean of amount_usd/amount_src for each currency and saving the results to a dictionary for easy lookup. The missing values in amount_src are filled by multiplying the values derived for the exchange rate and the source_currency column
