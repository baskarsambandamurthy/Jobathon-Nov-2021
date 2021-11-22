# Jobathon-Nov-2021

## Score
- Private LB Rank:  142
- Private LB Score:  0.6693
- Public LB Rank: 105
- Public LB Score:  0.6839

## Approach

1. `LastWorkingDate` field is used to determine whether the employee has resigned from the company or not. When the value is not null it means that employee has resigned from the company. A new target field name `is_resigned` created for every record in the train set using this `LastWorkingDate` field.

2. Target converted from current month whether employee resigned or not to whether employee will resign in next 6 monthts. This is achieved by copying the recent reporting employee resignation status record value to recent 6 months. Suppose if the recent reporting date contains employee resignation for the month as 1 , then recent 6 months reporting date also populated with 1 and similarly for the value 0. This helps in data scarcity in test data.

3. Since test set does not have employee information, the below information of the employees are simply copied from train set and for the remaining information which varies for every month, Lag features are created. 

4. Instead of KFold or StratifiedKFold 

## Cross Validation
There are 2 possible approaches of cross validation for this dataset.
- To use `StratifiedKFold` cross validation as there is an high class imbalance in the target column "redemption_status". (i.e) only less than 1% of data has target value of 1.

> **Stratification**: It  is a technique where we rearrange the data in a way that each fold has a good representation of the whole dataset. It forces each fold to have at least m instances of each class. This approach ensures that one class of data is not overrepresented especially when the target variable is unbalanced.

- To perform `time series` based validation
> **Time series cross-validation**:  In this validation,  validation data is determined based on time period and not random. So specific recent periods would be used as validation set and we consider the previous period’s data as the training set.

Without time series based validation, if StratifiedKFold CV was used, then CV score used to shoot up while test score would remain around the same score and this is because of target leak in CV since it uses the future transactions and hence due to this overfitting, there may not be much of improvement in test score.
So, it has been decided to use the **time series based cross validation** .


## Preprocessing

## Model training

## Future scope of improvements:
