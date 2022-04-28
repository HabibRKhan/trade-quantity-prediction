# Predicting Trade Quantities
International Merchandise Trade Statistics (IMTS) data contain many **missing and outliers quantity/net weight** information. As an example, among 128,675 records of Rice (HS 100630) Imports between 2013 and 2017, around 7% are missing and 13% are outliers.

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/rice-Outlier-share.PNG" width="30%" height="30%">

**The objective of this analysis is to find a suitable model to estimate these missing quantities.**

It is obvious that the Trade Value will have high correlation with the quantity. We will look further at factors which can explain variations in unit values (value/quantity) and use these factors to improve estimation.

A visual PowerBI report of the analysis can be found [here](https://app.powerbi.com/view?r=eyJrIjoiM2FmYzBkZjQtZDgyZi00MzI1LWE4YjMtMjEwYThlMzEzMjRhIiwidCI6IjBmOWUzNWRiLTU0NGYtNGY2MC1iZGNjLTVlYTQxNmU2ZGM3MCIsImMiOjh9&pageName=ReportSection)

## Determinants
Trade data usually come with below variables:
- Period
- Frequancy: Annual, Monthly
- Flow: Import/Export
- Commodity code: usually Harmonised System (HS) 6 digit codes
- Reporter: Country reporting the trade
- Partner: Country reporter is importing from/exporting to
- CIF value: Trade value including cost of freight and insurance (usually for imports)
- FOB value: Free-on-board trade value (usually exports)
- Net Weight: in Kg
- Supplementary quantity: quantity in number of items, cubic meter, litres, etc.
- Supplementary quantity unit
- Customs Procedure Codes (CPC): roughly, what's the import/export for
- Mode of Transport: of the import/export

The list of variables can be slightly different depending on the Reporter.

In addition to these, Standard country or area codes for statistical use or [M49](https://unstats.un.org/unsd/methodology/m49/), was merged with the data to get Regions, Subregions, and Development levels (Developing vs Developed) of each Reporter and Partner.

Unit values were calculated as value/net weight or value/quantity depending on the commodity. The analysis was run on Imports data of three different commodities:
- **Rice** (HS 100630)
- **Cars** (1500-3000cc, HS 870323)
- **Iron ores** (HS 260111)

For sake of simplicity, the PowerBI report and below sections will use the Rice data only. The Model comparison section will discuss all three commodities.

### Period
Period is a significant predictor of unit values. The ANOVA test shows this.

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/period.PNG" width="100%" height="100%">

### Regions and Subregions
Regions and subregions for bothe Reporters and Partners were significant. However, it was observed that Partner Subregion can account for more variations than any other variables.

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/repRegion.PNG" width="100%" height="100%">

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/partReg.PNG" width="100%" height="100%">

### Development Level
Although significant, Development level was not a strong predictor.

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/devLevel.PNG" width="100%" height="100%">

In addition to these, GDP per capita was examined for correlation with median unit values. No pattern was found.

## Models
Below models were used to estimate missing and outlier quantities. For some of them, unit values were estimated first which was converted to quantities by multiplying with values.

- **SUV**: Current practice. Quantities are estimated with global median values of given commodities after removing outliers.
- **Subregional SUV:** Similar to SUV but the medians were calculated per partner subregion.
- **Cluster SUV:** 4 clusters were fitted to partner countries using K-means and medians were calculated per partner cluster to estimate quantities from. Choice of the number of clusters was tricky. Usually 4-6 clusters returned the best fit for training data but 3-4 clusters were optimal for test data as that prevented overfitting.
- **Univariate regression (UNI):** Value on quantity.
- **Bivariate regression (BI):** Value and partner subregion on quantity.
- **Hierarchical regression (HI):** Value on weight for each partner subregion.
- **Cluster regression (Clust):** Value on weight for each cluster.
- **Multivariate regression (All):** Value, period, partner subregion, partner development level, reporter subregion, reporter development level on quantity

## Models Comparison
The data was randomly split into 80/20 training and test set. The models were fitted on the training set and predictions were made for the test set. For each of the commidities, the script was run 3 times as the random splitting of train and test would cause some variation.

The models were compared across three criteria:
**Mean deviation:** of fitted quantities from reported quantities in the test data.
**Variance of deviation:** of fitted quantities from reported quantities in the test data.
**Proportion of negative quantities:** among fitted quantities in the test data.

So in all three of these, **less is better**. Below image shows result for one of these runs:

<img src="https://github.com/HabibRKhan/trade-quantity-prediction/blob/master/images/modelCompare.PNG" width="100%" height="100%">

## Summary
So far, it seems, simpler models perform better. Subregional SUV, Cluster SUV and SUV give the best prediction in most cases. In some cases the Hierarchical model combining clustering and regression performs well. One thing is clear, a model with all possible predictor (Model All) is the worst as it evidently leads to serious overfitting.

### Next Steps
This analysis needs to be repeated for exports data to see if the dynamics are similar there. Also, Neural Network Models should be explored to see how well can they predict the quantities.
