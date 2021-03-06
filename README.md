

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

pd.set_option('display.max_columns', None)
```

For this exercise we will work through the different steps of a linear regression workflow. The notebook will walk you through building a first simple model and improving upon that model by stepwise iteration.

### 1. First Simple Model
- Load in the dataset: inspect the overall shape, duplicate entries, and na's.
- Identify the continuous target variable
- Perform Initial EDA: correlation plots
- Build a FSM (First Simple Model) with statsmodels/Build a FSM with sklearn
- Check the assumptions of linear regression  

### 2. Iterate: Build a better model - Add another numerical feature
- Add another feature, this time with high negative correlation, and fit the model
- Compare metrics and interpret coefficients
- Check the assumptions

### 3. Iterate: Build a better model - Add a categorical feature
- Add a categorical variable 
- Compare metrics and interpret coefficients
- Check the assumptions once-again

### 4. Appendix
- Next steps

## The Dataset
We will use a dataset from [Kaggle](https://www.kaggle.com/kumarajarshi/life-expectancy-who). It contains data collected by the WHO about life expectancy and potentially-related factors.  The information is aggregated on a per-country per-year basis.

The following questions have been posed. Read them and keep them in your mind when building your model.  We will reference them as we proceed through the workflow.

1. Do various predicting factors which have been chosen initially really affect life expectancy? Which predicting variables actually affect life expectancy?
2. Should a country having a lower life expectancy value(<65) increase its healthcare expenditure in order to improve its average lifespan?
3. How do infant and adult mortality rates affect life expectancy?
4. Does life expectancy have positive or negative correlation with eating habits, lifestyle, exercise, smoking, drinking alcohol etc.
5. What is the impact of schooling on the lifespan of humans?
6. Does Life expectancy have positive or negative relationship with drinking alcohol?
7. Do densely populated countries tend to have lower life expectancy?
8. What is the impact of immunization coverage on life Expectancy?

# 1. FSM

### Load in the dataset and check the overall shape



```python
# load in the dataset
df = None
```


```python
# How many records are in the data set?
records = None
```


```python
# How many columns are in the dataset?
columns = None
```


```python
# Check for duplicate entries
# Your answer here
```


```python
# Check for na's (just look to get an idea; don't drop or impute yet)
# Your answer here
```


```python
#__SOLUTION__
# 0. Load in the dataset and check the overall shape

# load in the dataset
df = pd.read_csv('data/life_expectancy.csv')

# How many records are in the data set?
records = df.shape[0]

# How many columns are in the dataset?
columns = len(df.columns)

# Check for duplicate entries
df.duplicated().sum()

# Check for na's (just look to get an idea; don't drop or impute yet)
df.isna().sum()

```

### What does a row in the dataframe represent?



```python
# Your answer here
```


```python
#__SOLUTION__
"""
Each row represents a *year* of a *country's* health data.
"""
```

### Identify the continous target variable


```python
#__SOLUTION__
# Identify the continuous target variable of interest
df['Life expectancy ']


```

If you had problems isolating that variable, don't worry.  That is expected! 
There can be odd, burdensome inconsistencies in naming of data.
Let's use our Python skills to wipe out the naming inconsistencies.

### Clean up the column names. 
There are many ways to do this. One way of doing so, outlined below, uses the columns attribute of the dataframe.  Then, using a list comprehension or for loop, we can manipulate the column name strings using methods that come with the string class.


```python
# 1. Gather column names into a variable
columns = None

# 2. Strip whitespace from the ends
columns = None

# 3. Replace white space with underscores
columns = None

# 4. Make all columns characters lowercase
columns = None

# 5. Reset the column names of the dataframe
# df.columns = columns
```


```python
# Check to make sure the changes are reflected in the dataset
# df.head()
```


```python
# Lastly, to make things easier to interpet, set the target to column index 0

```


```python
#__SOLUTION__
# Lastly, to make things easier to interpet, set the target to column index 0

cols = list(df.columns)
cols = [cols[3]] + cols[:3] + cols[4:]
df = df[cols]

```


```python
#__SOLUTION__

# 1. Gather column names into a variable
columns = df.columns

# 2. Strip whitespace from the ends
columns = [column.strip() for column in columns]

# 3. Replace white space with underscores
columns = [column.replace(' ', '_') for column in columns]

# 4. Make all columns characters lowercase
columns = [column.lower() for column in columns]

# 5. Reset the column names of the dataframe
df.columns = columns
```


```python
# Revisit the continuous target variable.  
# Explore it a bit.  Plot a histogram of its distribution as well as a boxplot
```


```python
#__SOLUTION__
# Revisit the continuous target variable.  
# Explore it a bit.  Plot it's distribution and boxplot

fig, ax = plt.subplots(2,1, figsize=(10,5))
sns.distplot(df.life_expectancy, ax = ax[0])
sns.boxplot(df.life_expectancy, ax= ax[1])

ax[0].set_title('Distribution of Target Variable: Life Expectancy');
```


```python
df.sort_values('life_expectancy').head(10)
df[df.country=='Haiti']
```


```python
# Describe the distribution of the target
# Look at the min value? What happened in Haiti in 2010?
```


```python
#__SOLUTION__
print(f'''Generally normal with a left skew.  
            Mean of {round(df.life_expectancy.mean(),2)}
             Median of {round(df.life_expectancy.median(),2)}
             Skew: {round(stats.skew(df.life_expectancy.dropna()), 2)}
             ''')
```

## Perform Initial EDA

There are a lot of variables here!  Let's look at a correlation matrix to see which ones might be the most useful.  (Here we are looking for variables that are highly correlated with the target variable, but not highly correlated with other input variables)


```python
# create a correlation matrix
# first, just use the datafram .corr() method to output a numerical matrix

# Your answer here
```


```python
# Then pass the above code into Seaborn's heatmap plot

# Your answer here
```


```python
# Try adding the code in this cell to the mask attribute in the heatmap to halve the plot
mask = np.triu(np.ones_like(df.corr(), dtype=np.bool))

# Your answer here
```


```python
#__SOLUTION__
# create a correlation matrix
# first, just use the datafram .corr() method to output a numerical matrix
df.corr()

# Then pass the above code into Seaborn's heatmap plot
mask = np.triu(np.ones_like(df.corr(), dtype=np.bool))

fig, ax = plt.subplots(figsize=(15,10))
sns.heatmap(df.corr(), mask=mask)
```


```python
# Judging from the correlation matrix or the heatmap, which three features have the highest positive correlation? 

# Your answer here
```


```python
#__SOLUTION__
# From the heatmap, which features have high correlation? 
"Schooling, income_composition_of_resources, BMI"
```

### Use seaborn's pairplot function on the three features above plus life_expectancy.  
Note: we would usually start right off by using a pairplot, but because we have so many features, the pairplot would be unwieldy.


```python
# your code here
```


```python
#__SOLUTION__
high_correlation_df = df[['life_expectancy', 'schooling', 
                         'income_composition_of_resources', 'bmi']]

sns.pairplot(high_correlation_df)
```

Judging from the top row of the pairplot, one feature's correlation to the target is a bit fuzzier than the rest. 
Inspecting other cells of the pairplot, the other two features show covariance. 
Given those two insights, choose one feature to build the First Simple Model with. (Our FSM will be simple the target and one predictor).
Consider also whether choosing one of the positively correlated features above the others would help answer any of the question listed at the beginning of the notebook.



```python
#__SOLUTION__

"""
It looks like the correlation with BMI is a little fuzzier than the others, 
so let's exclude it for now.  
`Schooling` and `Income_Composition_of_Resources` are highly correlated with both life expectancy and each other, 
so let's only include one of them. 
`Schooling` seems like a good choice because it would allow us to answer Question 5.
"""

```

## FSM with Statsmodels



```python
from statsmodels.formula.api import ols
# Create a dataframe with only the target and the chosen high-positive corellation feature
fsm_df = None
```


```python
# For this FSM, simply dropnas.

# your code here
```


```python
# build the R-style formula. The format is "target~feature_1 + feature_2 + feature_3"
formula = None
```


```python
# Fit the model on the dataframe composed of the two features
fsm = ols(formula=formula, data=fsm_df).fit()
```


```python
#__SOLUTION__
from statsmodels.formula.api import ols
# Create a dataframe with only the target and the chosen high-positive corellation feature

fsm_df = df[["schooling", "life_expectancy"]].copy()
fsm_df.dropna(inplace=True)

# build the R-style formula. The format is "target~feature_1 + feature_2 + feature_3"

formula = "life_expectancy ~ schooling"

# Fit the model on the dataframe composed of the two features

fsm = ols(formula=formula, data=fsm_df).fit()
```


```python
# Use the summary() method on the fsm varaible to print out the results of the fit.
fsm.summary()
```


```python
# The object also has attributes associated with the ouput, such as: rsquared, and params.
# save those values to the variables below.

rsquared = None
params = None

print(f'Rsquared of FSM: {rsquared}')
print('----------')
print('Beta values of FSM:')
print(params)
```


```python
#__SOLUTION__
# The object also has attributes associated with the ouput, such as: rsquared, and params.
# save those values to the variables below.

rsquared = fsm.rsquared
params = fsm.params

print(f'Rsquared of FSM: {rsquared}')
print('----------')
print('Beta values of FSM:')
print(params)
              
```

Interpret the result of the FSM.  What does the R Squared tell you? Remember the formula for:

$\Large R^2 = 1 - \frac{SSE}{SST}$

Also, interepret the coefficients.  If we increase the value of our independent variable by 1, what does it mean for our predicted value?

What will our model predict the value of Life Expectancy to be for a country with 0 years of school on average?


```python
# Your answer here
```


```python
#__SOLUTION__
'''
Our R_2 is not too bad. We are only explaining about 57% of the variance in life expectancy, but we only have one feature so far and it's statistically significant at an alpha of 0.05.

We could stop right now and say that according to our model:

 - A country with zero years of schooling on average is expected to have a life expectancy of 44.1 years
 - For each additional average year of schooling, we expect life expectancy to increase by 2.1 years
 
'''
```

# Check the assumptions of Linear Regression

#### 1. Linearity

Linear regression assumes that the input variable linearly predicts the output variable.  We already qualitatively checked that with a scatter plot.  But it's also a good idea to use a statistical test.  This one is the [Rainbow test](https://www.tandfonline.com/doi/abs/10.1080/03610928208828423) which is available from the [diagnostic submodule of StatsModels](https://www.statsmodels.org/stable/generated/statsmodels.stats.diagnostic.linear_rainbow.html#statsmodels.stats.diagnostic.linear_rainbow)


```python
from statsmodels.stats.diagnostic import linear_rainbow, het_breuschpagan
from statsmodels.stats.outliers_influence import variance_inflation_factor

rainbow_statistic, rainbow_p_value = linear_rainbow(fsm)
print("Rainbow statistic:", rainbow_statistic)
print("Rainbow p-value:", rainbow_p_value)
```

The null hypothesis is that the model is linearly predicted by the features, alternative hypothesis is that it is not.  Thus returning a low p-value means that the current model violates the linearity assumption.

#### 2. Normality

Linear regression assumes that the residuals are normally distributed.  It is possible to check this qualitatively with a Q-Q plot.  The fit model object has an attribute called resid, which is an array of the difference between predicted and true values.  Store the residuals in the variable below, show the qq plot, and interepret. You are looking for the theoretical quantiles and the sample quantiles to line up.


```python
# Create a qq-plot

fsm_resids = None

import statsmodels.api as sm
sm.qqplot(fsm_resids)
```


```python
#__SOLUTION__
# Create a qq-plot

fsm_resids = fsm.resid

import statsmodels.api as sm
sm.qqplot(fsm_resids)
```

Those qqplots don't look so good in the upper right corner. To pass a visual test, the qq should be a straight line.

The [Jarque-Bera](https://en.wikipedia.org/wiki/Jarque%E2%80%93Bera_test) test is performed automatically as part of the model summary output, labeled **Jarque-Bera (JB)** and **Prob(JB)**.

The null hypothesis is that the residuals are normally distributed, alternative hypothesis is that they are not.  
What does the JB score output indicate. Does it support the qq-plot?


```python
# Your answer here
```


```python
#__SOLUTION__
'''The JB score has a low p-value means that the current model violates the normality assumption. 
That supports the qq visual with the crooked tail.'''
```

#### 3. Homoscadasticity

Linear regression assumes that the variance of the dependent variable is homogeneous across different values of the independent variable(s).  We can visualize this by looking at the predicted life expectancy vs. the residuals.




```python
# Use the predict() method now available to be called from the fsm variable to store the predictions
y_hat = None
```


```python
# plot y_hat against the residuals (stored in fsm_resids) in a scatter plot

# Your code here
```

Interepret the result. Do you see any patterns that suggest that the residuals exhibit heteroscedasticity?



```python
#__SOLUTION__

y_hat = fsm.predict()

fig, ax = plt.subplots()

ax.scatter(y_hat, fsm_resids)
```


```python
#__SOLUTION__
'''
Just visually inspecting this, it seems like our model over-predicts life expectancy 
between 60 and 70 years old in a way that doesn't happen for other age groups.  
Plus we have some weird-looking data in the lower end that we might want to inspect.  
Maybe there was something wrong with recording those values, 
or maybe there is something we can feature engineer once we have more independent variables.
'''
```

Let's also run a statistical test.  The [Breusch-Pagan test](https://en.wikipedia.org/wiki/Breusch%E2%80%93Pagan_test) is available from the [diagnostic submodule of StatsModels](https://www.statsmodels.org/stable/generated/statsmodels.stats.diagnostic.het_breuschpagan.html#statsmodels.stats.diagnostic.het_breuschpagan)


```python
# If you chose schooling, this should run without error
lm, lm_p_value, fvalue, f_p_value = het_breuschpagan(fsm_resids, fsm_df[["schooling"]])
print("Lagrange Multiplier p-value:", lm_p_value)
print("F-statistic p-value:", f_p_value)
```

The null hypothesis is homoscedasticity, alternative hypothesis is heteroscedasticity.  
What does the p-value returned above indicate?


```python
#__SOLUTION__
'''Thus returning a low p-value means that the current 
model violates the homoscedasticity assumption'''

```

#### 4. Independence

The independence assumption means that the independent variables must not be too collinear.  Right now we have only one independent variable, so we don't need to check this yet.

## Train a model with sklearn
The sklearn interface is simpler than Statsmodels, but it does not give us the super helpful statsmodel output.  We will, however, use its syntax consistently with other algorithms.  


```python
from sklearn.linear_model import LinearRegression

# fit an sklearn model
#instantiate a linear regression object 
lr = None
```


```python
# split the data into target and features
y = None
X = None
```


```python
# Call .fit from the linear regression object, and feed X and y in as parameters
# Your code here
```


```python
# lr has a method called score.  Again, feed in X and y, and read the output. Save it in the variable score. 
# What is that number?  Compare it to statsmodels. 
score = None
```


```python
# lr also has attributes coef_ and intercept_. Save and compare to statsmodels
beta = None
intercept = None
```


```python
#__SOLUTION__
from sklearn.linear_model import LinearRegression

# fit an sklearn model
#instantiate a linear regression object 
lr = LinearRegression()

# split the data into target and features
y = fsm_df.life_expectancy
X = fsm_df.drop('life_expectancy', axis=1)

# Call .fit from the linear regression object, and feed X and y in as parameters
lr.fit(X,y)

# lr has a method called score.  Again, feed in X and y, and read the output. Save it in the variable score.  What is that number?  Compare it to statsmodels. 
score = lr.score(X,y)
# that is the r_2.  It is the same as the Statsmodels R_2

# lr also has attributes coef_ and intercept_. Save and compare to statsmodels
beta = lr.coef_
intercept = lr.intercept_
#sklearn calculates the same coeficients and intercepts as statmodels.

```

# 2. Iterate: Build a better model - Add another numerical feature

## Adding Features to the Model

So far, all we have is a simple linear regression.  Let's start adding features to make it a multiple regression.

Let's repeat the process of the highly positively correlated variables, but this time with the highly negatively correlated variables (based on looking at the correlation matrix)


```python
negatively_correlated_cols = [
    'life_expectancy',
    'adult_mortality',
    'hiv/aids',
    'thinness__1-19_years',
    'thinness_5-9_years'
]
negatively_correlated_df = df[negatively_correlated_cols]
sns.pairplot(negatively_correlated_df);
```

`adult_mortality` seems most like a linear relationship.  Also, the two thinness metrics seem to be providing very similar information, so we almost certainly should not include both

Let's proceed with adult mortality.


```python
# Create another dataframe containing our three features of interest
model_2 = None
```


```python
# Drop na's across all columns
```


```python
# save the R-like formula into the variable
formula = None
```


```python
# train the model like we did above
model_2
```


```python
# print out the summary table
# Your code here
```


```python
#__SOLUTION__

# Create another dataframe containing our three features of interest
model_2_df = df[['life_expectancy', 'schooling', 'adult_mortality']].copy()

# Drop na's across all columns
model_2_df.dropna(inplace=True)

# save the R-like formula into the variable
formula = 'life_expectancy~schooling+adult_mortality'

# train the model like we did above
model_2 = ols(formula=formula, data=model_2_df).fit()

# print out the summary
model_2.summary()

```

### Did the r_2 improve? 
Your answer here


```python
#__SOLUTION__
'Adding another feature improved the r-squared from 0.565 to 0.714'
```

### Now check the assumptions like we did above.


```python
## Linearity

# Your answer here

```


```python
#__SOLUTION__
rainbow_statistic, rainbow_p_value = linear_rainbow(model_2)
print("Rainbow statistic:", rainbow_statistic)
print("Rainbow p-value:", rainbow_p_value)

'Assuming an alpha of 0.05, we are no longer violating the linearity assumption (just barely)'
```


```python
## Normality

# Your answer here

```


```python
#__SOLUTION__
'''The Jarque-Bera (JB) output has gotten worse. We are still violating the normality assumption.'''
```


```python
## Homoscadasticity

# Your answer here

```


```python
#__SOLUTION__
y_hat = model_2.predict()
model_2_resids = model_2.resid

fig4, ax4 = plt.subplots()
ax4.set(xlabel="Predicted Life Expectancy",
        ylabel="Residuals (Actual - Predicted Life Expectancy)")
ax4.scatter(x=y_hat, y=model_2_resids, color="blue", alpha=0.2)
```


```python
#__SOLUTION__
lm, lm_p_value, fvalue, f_p_value = het_breuschpagan(y-y_hat, model_2_df[["schooling", "adult_mortality"]])
print("Lagrange Multiplier p-value:", lm_p_value)
print("F-statistic p-value:", f_p_value)

'''Both visually and numerically, we can see some improvement. 
But we are still violating this assumption to a statistically significant degree.'''
```

## Independence

You might have noticed in the regression output that there was a warning about the condition number being high. The condition number is a measure of stability of the matrix used for computing the regression (we'll discuss this more in the next module), and a number above 30 can indicate strong multicollinearity. Our output is way higher than that.

A different (more generous) measure of multicollinearity is the variance inflation factor. It is available from the outlier influence submodule of StatsModels.

Run the code below:


```python
rows = model_2_df[["schooling", "adult_mortality"]].values

vif_df = pd.DataFrame()
vif_df["VIF"] = [variance_inflation_factor(rows, i) for i in range(2)]
vif_df["feature"] = ["schooling", "adult_mortality"]

vif_df
```

A "rule of thumb" for VIF is that 5 is too high.  Given the output above, it's reasonable to say that we are not violating the independence assumption, despite the high condition number.

### 3. Iterate: Build a better model - Add a categorical feature


This is less realistic than the previous steps, but is good for demonstartion purposes.

In this dataset, we have a lot of numeric values (everything in that correlation matrix), but there are a few that aren't.  One example is `Status`


```python
# We have created a dataframe with the "life_expectancy", "schooling", "adult_mortality", "status"] columns
model_3_df = df[["life_expectancy", "schooling", "adult_mortality", "status"]].copy()

# Drop NA's

# your code here
```


```python
# Inspect value counts  of the status column

# Your code here

```


```python
#__SOLUTION__

# Create a dataframe with the "life_expectancy", "schooling", "adult_mortality", "status"] columns
model_3_df = df[["life_expectancy", "schooling", "adult_mortality", "status"]].copy()

# Drop NA's
model_3_df.dropna(inplace=True)

# Inspect value counts  of the status column
model_3_df["status"].value_counts()




```


```python
# Check out what Seaborn's catplot does
# https://seaborn.pydata.org/generated/seaborn.catplot.html

# Plot status vs life expectancy.  Choose a kind of plot to pass into the kind parameter

# Your code here
```


```python
# Plot status vs life expectancy.  Choose a kind of plot to pass into the kind parameter
#__SOLUTION__
sns.catplot(x="status", y="life_expectancy", data=model_3_df, kind='box')
```

It looks like there is a difference between the two groups that might be useful to include

There are only two categories, so we only need a `LabelEncoder` that will convert the labels into 1s and 0s.  If there were more than two categories, we would use a `OneHotEncoder`, which would create multiple columns out of a single column.


```python
from sklearn.preprocessing import LabelEncoder

# instantiate and instance of LabelEncoder
label_encoder = None
```


```python
# Pass the "status" column of the model_3_df to the fit_transform() method of the Label Encoder
status_labels = None
```


```python
#__SOLUTION__
from sklearn.preprocessing import LabelEncoder

# instantiate and instance of LabelEncoder
label_encoder = LabelEncoder()

# Pass the "status" column of the model_3_df to the fit_transform() method of the Label Encoder
status_labels = label_encoder.fit_transform(model_3_df["status"])

```


```python
# Run the code below.  The category Developing/Developed has been transformed to a binary
np.unique(status_labels, return_counts=True)
```


```python
# Run the code below to see the classes associated with 1 and 0
label_encoder.classes_
```

This is telling us that "Developed" is encoded as 0 and "Developing" is encoded as 1.  This means that "Developed" is assumed at the intercept.


```python
# Add the status labels array to the model_df as a column 
model_3_df["status_encoded"] = None
```


```python
# Drop the status column

# your code here
```


```python
#__SOLUTION__
# Add the status labels array to the model_df as a column 
model_3_df["status_encoded"] = status_labels
model_3_df.drop("status", axis=1, inplace=True)
```


```python
# Fit the 3rd model

# assign the new formula

formula=None
```


```python
# fit the new model
model_3 = None
```


```python
# print the summary
model_3.summary()
```


```python
#__SOLUTION__
# Fit the 3rd model

# assign the new formula

formula="life_expectancy~" + "+".join(model_3_df.iloc[:,1:].columns)

# fit the new model
model_3 = ols(formula=formula, data=model_3_df).fit()

# print the summary
model_3.summary()
```

### Third Model Evaluation

Did the R_squared improve?

Your answer here


```python
#__SOLUTION__
# Did the R_squared improve
"Adding another feature improved the r-squared a tiny bit from 0.714 to 0.718"
```

# Let's look at the model assumptions again

#### Linearity


```python
rainbow_statistic, rainbow_p_value = linear_rainbow(model_3)
print("Rainbow statistic:", rainbow_statistic)
print("Rainbow p-value:", rainbow_p_value)
```


```python
# Did linearity improve

```


```python
#_SOLUTION__
'Another small improvement'
```

#### Normality
Did our errors become more normally distributed?

Your answer here


```python
#__SOLUTION__
#### Normality

'''
The **Jarque-Bera (JB)** output has gotten slightly better.  
But we are still violating the normality assumption.
'''


```

#### Homoscadasticity

Did our homoscadasticity improve?


```python
model_3_resids = None
y_hat = None

fig, ax = plt.subplots()

ax.scatter(y_hat, model_3_resids)
```


```python
lm, lm_p_value, fvalue, f_p_value = het_breuschpagan(y-y_hat, model_3_df[["schooling", "adult_mortality", "status_encoded"]])
print("Lagrange Multiplier p-value:", lm_p_value)
print("F-statistic p-value:", f_p_value)
```


```python

#__SOLUTION__

model_3_resids = model_3.resid
y_hat = model_3.predict()

fig, ax = plt.subplots()

ax.scatter(y_hat, model_3_resids)
```


```python
#__SOLUTION__
lm, lm_p_value, fvalue, f_p_value = het_breuschpagan(y-y_hat, model_3_df[["schooling", "adult_mortality", "status_encoded"]])
print("Lagrange Multiplier p-value:", lm_p_value)
print("F-statistic p-value:", f_p_value)
```


```python
#__SOLUTION__
'''This metric got worse, although the plot looks fairly similar'''
```

#### Independence


```python
rows = model_3_df[["schooling", "adult_mortality", "status_encoded"]].values

vif_df = pd.DataFrame()
vif_df["VIF"] = [variance_inflation_factor(rows, i) for i in range(3)]
vif_df["feature"] = ["schooling", "adult_mortality", "status_encoded"]

vif_df
```

What does the above output tell you?

Your answer here


```python
#__SOLUTION__
"""The VIF metrics are getting higher, which means that there is stronger multicollinearity.  
But we have still not exceeded the threshold of 5."""
```

Below, you will find an example summary of how one might use the linear regression models shown above to address the questions posed at the beginning of the notebook.

# Summary
We started with a baseline model where the only input feature was Schooling. Our baseline model had an r-squared of 0.565. This model violated the linearity (p < 0.001), normality (p < 0.001), and homoscadasticity (p < 0.001) assumptions of linear regression. The independence assumption was met by default because there was only one input feature.

The final model for this lesson had three input features: Schooling, Adult_Mortality, and Status_Encoded. It had an r-squared of 0.718. This model did not violate the linearity assumption (p = 0.084), but it did violate the normality (p < 0.001) and homoscedasticity (p < 0.001) assumptions. Based on the variance inflaction factor metric, it did not violate the independence assumption.

We are able to address the following questions from above:

1. Do various predicting factors which have been chosen initially really affect the Life expectancy? What are the predicting variables actually affecting the life expectancy?

With only 3 features we are able to explain about 71% of the variance in life expectancy. This indicates that these factors truly are explanatory. More analysis is required to understand how much additional explanatory power would be provided by incorporating additional features into the model.

3. How do Infant and Adult mortality rates affect life expectancy?

So far we have only investigated adult mortality. The adult mortality rate ("probability of dying between 15 and 60 years per 1000 population") has a negative correlation with life expectancy. For each increase of 1 in the adult mortality rate, life expectancy decreases by about .03 years.

5. What is the impact of schooling on the lifespan of humans?

In our latest model, we find that each additional year of average schooling is associated with 1.4 years of added life expectancy. However it is challenging to interpret whether it is schooling that is actually having the impact. Schooling is highly correlated with Income_Composition_of_Resources ("Human Development Index in terms of income composition of resources") so it is very possible that schooling is the result of some underlying factor that also impacts life expectancy, rather than schooling impacting life expectancy directly.

### 4. Appendix


Things we have not done in this lesson, but that you should consider in your project:  

- More robust cleaning (possible imputation of missing values, principled exclusion of some data)  
- Feature scaling  
- Nearest-neighbors approach (requires more complex feature engineering)  
- Pulling information from external resources  
- Removing independent variables if you determine that they are causing too high of multicollinearity  
- Setting up functions so the code is not so repetitive  
