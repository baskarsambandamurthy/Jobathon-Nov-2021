# Jobathon-Nov-2021

## Score
- Private LB Rank:  142
- Private LB Score:  0.6693
- Public LB Rank: 105
- Public LB Score:  0.6839

## Approach

#### Target Analysis

1. Target field is not directly available in the dataset. The `LastWorkingDate` field is used to determine whether the employee has resigned from the company or not. When the value is not null means that employee has resigned from the company. A new target field name `is_resigned` created for every record in the train set using this `LastWorkingDate` field.

2. Dataset contains the target information for the current month (i.e) Whether employee has resigned in the current month or not. But the problem requires that every employee has only one target specifying whether the employee would resign in next 6 months or not. **Hence the target variable (`is_resigned`) in the training set is converted from current month to next 6 months.** This is achieved by filling the recent reporting employee resignation status record value to recent 6 months. <br>
Suppose if the recent reporting date contains employee resignation for the month as 1 , then recent 6 months reporting date also populated with 1 and similarly for the value 0. This helps in data scarcity in test data.

#### Key Features

1. Training Set comprises of 2 kind of information for each employee based on whether the information varies over every reporting date
    - Static Fields for given employee
      - _Gender_ 
      - _Education_Level_ 
      - _Joining Designation_ 
    - Dynamic which might vary for different reporting dates of the employee
      - _Age_ 
      - _Total Business Value_ 
      - _Designation_ 
      - _Quarterly Rating_
      - _Salary_
  
2. Since test set does not have employee information on current reporting date, the static fields of the employees are simply copied from the corresponding data in train set and for the remaining dynamic fields which varies for every month, **Lag features** are created

#### Cross Validation Technique
There are 2 possible approaches of cross validation for this dataset.
- To use `StratifiedKFold` cross validation as there is an high class imbalance in the target column.

> **Stratification**: It  is a technique where we rearrange the data in a way that each fold has a good representation of the whole dataset. It forces each fold to have at least m instances of each class. This approach ensures that one class of data is not overrepresented especially when the target variable is unbalanced.

- To perform `time series` based validation
> **Time series cross-validation**:  In this validation,  validation data is determined based on time period and not random. So specific recent periods would be used as validation set and we consider the previous period???s data as the training set.

Without time series based validation, if StratifiedKFold CV was used, then CV score used to shoot up while test score would remain around the same score and this is because of target leak in CV since it uses the future transactions and hence due to this overfitting, there may not be much of improvement in test score.
So, it has been decided to use the **time series based cross validation** .


## Preprocessing

- A new field Number of Months since joined field (`months_since_joined`) is generated by computing difference between reporting date field (i.e) `MMM-YY` and joining date field `JoiningDate`. This field `months_since_joined` is used as one of the feature for training and also it is used to convert the target format of current month to the target format of next 6 months.
- The following categorical fields containing string data is encoded to numeric using Label encoding method
    - `Gender`,`Education_Level`
- The categorical field `City` is encoded using one hot encoding technique since this field has lot of values.
- Lag features are generated for the dynamic fields on combined train and test dataset so that test will also have the information shifted from train. These lag features are generated for each employee. There are 3 set of lag features created for corresponding last 3 months.

## Model training
<br>

| Method      | Which one Used in the Model  |
|----------------------|-------------------------------|
|Cross Validation        |Time Series Validation 
|Algorithm        | Light GBM which uses Boosting technique for training. Light GBM is chosen since it can work well with nulls (in lag features) and very fast and could perform very good prediction
|Type        |Classification- Binary
|Target        |`is_resigned` (i.e) For any given employee record, train and predict whether employee record resigns in next 6 months
|Features used        |'Gender', 'Education_Level', 'Joining Designation', 'months_since_joined', 3 set of lag features for each of Age, Total Business Value,Quarterly Rating,Salary,Designation, city one hot encoded fields

<br>

There are 2 models used in this solution and both models uses the same above categories mentioned and the difference lies with the training set used in the models.

#### First Model
  - Here validation set is for 2 quarters ranging from 07-Jul-2017 till 12-Dec-2017. 
  - Inorder to have same format as testset record, only one record per employee is maintained in validation set (i.e) For each employee, only the record pertaining to Jul-2017 month is retained.
  - Training set ranges from starting report date of the dataset till Jun-2017 for each employee

#### Second Model
  - Here no validation set used
  - Full Training set i.e reporting date till Dec-2017 for each employee

#### Why 2 models are used ?
  - First model is trained with the training set till specific reporting month and corresponding validation set and then best iteration is achieved on the best score of the validation set. 
  - But this model would be best suitable for given validation set and since the training set comprises only around 60% of data in this case, this is not the complete learning for making test predictions. 
  - So another model is used with training set as full set (i.e till Dec-2017) and without any validation set. But since the boosting algorithms, training error is always reducing for every iteration, the best iteration achieved on the previous model is used to avoid overfitting of the training data on this model.So both the models are needed for test predictions.
  - For test predictions, the full train model is used to perform predictions and the labels are converted using train probability optimal cutoff. Refer to below section regarding optimal cutoff.


#### Label prediction for imbalanced target class
  - Since the target classes are of the imbalanced form (i.e) target 1 has very less instances compared to target 0, converting probabilities to labels using 0.5 cutoff may not be the perfect.
  - Instead based on precision recall curve, the probability cutoff at which the maximum F1 score is obtained for the validation set. This optimum cutoff is used to convert the labels of both validation set and also the test set.
