import pandas as pd
import statsmodels.api as sm
from statsmodels.sandbox.regression.gmm import IV2SLS
import numpy as np

# Step 1: Load the data from Excel file
file_path = "Project1Data.xlsx"
data = pd.read_excel(file_path)
print(data.head())

      maand  year  month    qu  cprice  tprice  oprice   income  q1  q2  q3  \
0  Jan 1990  1990      1  0.55   12.12    18.6     1.0  1640.87   1   0   0   
1  Feb 1990  1990      2  0.65   12.12    18.6     1.0  1538.60   1   0   0   
2  Mar 1990  1990      3  0.66   12.12    18.6     1.0  1680.93   1   0   0   
3  Apr 1990  1990      4  0.66   12.12    18.6     1.0  1656.20   0   1   0   
4  May 1990  1990      5  0.64   12.12    18.6     1.0  1700.80   0   1   0   

   q4  bprice  wprice  
0   0    3.47   28.15  
1   0    3.40   28.15  
2   0    3.26   28.33  
3   0    3.46   28.49  
4   0    3.47   28.55  


#Step 2: Run 2SLS for demand  BASIC 
# 'cprice' is the endogenous variable for demand, predicted by IV
# Instruments (IV) for demand: 'bprice', 'wprice'
# No Control variables: CANNOT WORK!


# First Stage: Predict cprice using IVs (bprice, wprice)
# Prepare the independent variables for the first stage (instruments)
data['bprice_relative']=data['bprice']/data['oprice']
data['wprice_relative']=data['wprice']/data['oprice']
data['tprice_relative']=data['bprice']/data['oprice']
data['cprice_relative']=data['cprice']/data['oprice']
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
demand_iv = data[['bprice_relative', 'wprice_relative']]
demand_iv = sm.add_constant(demand_iv)  # Add constant


# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, demand_iv).fit()
predicted_cprice = first_stage_model.predict(demand_iv)

# Second Stage: Use predicted cprice to estimate the demand function
# Prepare demand function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
demand_X = pd.DataFrame({
    'predicted_cprice': predicted_cprice})
demand_X = sm.add_constant(demand_X)  # Add constant term
demand_y = data['qu']  # Dependent variable (coffee consumption)

# Second stage regression: Predict demand with predicted cprice
second_stage_model = sm.OLS(demand_y, demand_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Demand Function with Predicted cprice):")
print(second_stage_model.summary())
First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.939
Model:                            OLS   Adj. R-squared:                  0.938
Method:                 Least Squares   F-statistic:                     625.3
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           5.72e-50
Time:                        19:43:42   Log-Likelihood:                -53.165
No. Observations:                  84   AIC:                             112.3
Df Residuals:                      81   BIC:                             119.6
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
-----------------------------------------------------------------------------------
const              -3.9185      4.033     -0.972      0.334     -11.943       4.106
bprice_relative     1.8178      0.052     35.055      0.000       1.715       1.921
wprice_relative     0.3449      0.137      2.523      0.014       0.073       0.617
==============================================================================
Omnibus:                        2.851   Durbin-Watson:                   0.619
Prob(Omnibus):                  0.240   Jarque-Bera (JB):                2.318
Skew:                           0.206   Prob(JB):                        0.314
Kurtosis:                       3.702   Cond. No.                     2.35e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.35e+03. This might indicate that there are
strong multicollinearity or other numerical problems.

Second Stage Results (Demand Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.043
Model:                            OLS   Adj. R-squared:                  0.032
Method:                 Least Squares   F-statistic:                     3.722
Date:                Tue, 17 Sep 2024   Prob (F-statistic):             0.0572
Time:                        19:43:42   Log-Likelihood:                 95.727
No. Observations:                  84   AIC:                            -187.5
Df Residuals:                      82   BIC:                            -182.6
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
const                0.7997      0.062     12.930      0.000       0.677       0.923
predicted_cprice    -0.0092      0.005     -1.929      0.057      -0.019       0.000
==============================================================================
Omnibus:                       31.216   Durbin-Watson:                   1.715
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               69.260
Skew:                           1.315   Prob(JB):                     9.13e-16
Kurtosis:                       6.588   Cond. No.                         94.3
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.


#Step 2: Run 2SLS for demand  OPTION1
# 'cprice' is the endogenous variable for demand, predicted by IV
# Instruments (IV) for demand: 'bprice', 'wprice'
# Control variables:  'tprice' 


# First Stage: Predict cprice using IVs (bprice, wprice)
# Prepare the independent variables for the first stage (instruments)
data['bprice_relative']=data['bprice']/data['oprice']
data['wprice_relative']=data['wprice']/data['oprice']
data['tprice_relative']=data['bprice']/data['oprice']
data['cprice_relative']=data['cprice']/data['oprice']
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
demand_iv = data[['bprice_relative', 'wprice_relative','tprice_relative']]
demand_iv = sm.add_constant(demand_iv)  # Add constant


# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, demand_iv).fit()
predicted_cprice = first_stage_model.predict(demand_iv)

# Second Stage: Use predicted cprice to estimate the demand function
# Prepare demand function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
demand_X = pd.DataFrame({
    'predicted_cprice': predicted_cprice,
    'tprice': data['tprice']})
demand_X = sm.add_constant(demand_X)  # Add constant term
demand_y = data['qu']  # Dependent variable (coffee consumption)

# Second stage regression: Predict demand with predicted cprice
second_stage_model = sm.OLS(demand_y, demand_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Demand Function with Predicted cprice):")
print(second_stage_model.summary())
First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.939
Model:                            OLS   Adj. R-squared:                  0.938
Method:                 Least Squares   F-statistic:                     625.3
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           5.72e-50
Time:                        19:39:27   Log-Likelihood:                -53.165
No. Observations:                  84   AIC:                             112.3
Df Residuals:                      81   BIC:                             119.6
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
-----------------------------------------------------------------------------------
const              -3.9185      4.033     -0.972      0.334     -11.943       4.106
bprice_relative     0.9089      0.026     35.055      0.000       0.857       0.960
wprice_relative     0.3449      0.137      2.523      0.014       0.073       0.617
tprice_relative     0.9089      0.026     35.055      0.000       0.857       0.960
==============================================================================
Omnibus:                        2.851   Durbin-Watson:                   0.619
Prob(Omnibus):                  0.240   Jarque-Bera (JB):                2.318
Skew:                           0.206   Prob(JB):                        0.314
Kurtosis:                       3.702   Cond. No.                     2.17e+17
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The smallest eigenvalue is 1.57e-30. This might indicate that there are
strong multicollinearity problems or that the design matrix is singular.

Second Stage Results (Demand Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.080
Model:                            OLS   Adj. R-squared:                  0.058
Method:                 Least Squares   F-statistic:                     3.540
Date:                Tue, 17 Sep 2024   Prob (F-statistic):             0.0336
Time:                        19:39:27   Log-Likelihood:                 97.382
No. Observations:                  84   AIC:                            -188.8
Df Residuals:                      81   BIC:                            -181.5
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
const                0.1502      0.365      0.411      0.682      -0.576       0.877
predicted_cprice    -0.0167      0.006     -2.660      0.009      -0.029      -0.004
tprice               0.0389      0.022      1.804      0.075      -0.004       0.082
==============================================================================
Omnibus:                       36.648   Durbin-Watson:                   1.798
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              113.562
Skew:                           1.386   Prob(JB):                     2.19e-25
Kurtosis:                       7.977   Cond. No.                     1.00e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large,  1e+03. This might indicate that there are
strong multicollinearity or other numerical problems.

[5]
#Step 2: Run 2SLS for demand  OPTION2
# 'cprice' is the endogenous variable for demand, predicted by IV
# Instruments (IV) for demand: 'bprice', 'wprice'
# Control variables:  'tprice' 'q1' 'q2' 'q3'


# First Stage: Predict cprice using IVs (bprice, wprice)
# Prepare the independent variables for the first stage (instruments)
data['bprice_relative']=data['bprice']/data['oprice']
data['wprice_relative']=data['wprice']/data['oprice']
data['tprice_relative']=data['bprice']/data['oprice']
data['cprice_relative']=data['cprice']/data['oprice']
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
demand_iv = data[['bprice_relative', 'wprice_relative','tprice_relative','q1','q2','q3']]
demand_iv = sm.add_constant(demand_iv)  # Add constant


# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, demand_iv).fit()
predicted_cprice = first_stage_model.predict(demand_iv)

# Second Stage: Use predicted cprice to estimate the demand function
# Prepare demand function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
demand_X = pd.DataFrame({
    'predicted_cprice': predicted_cprice,
    'tprice': data['tprice'],
    'q1': data['q1'],
    'q2': data['q2'],
    'q3': data['q3']})
demand_X = sm.add_constant(demand_X)  # Add constant term
demand_y = data['qu']  # Dependent variable (coffee consumption)

# Second stage regression: Predict demand with predicted cprice
second_stage_model = sm.OLS(demand_y, demand_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Demand Function with Predicted cprice):")
print(second_stage_model.summary())
First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.941
Model:                            OLS   Adj. R-squared:                  0.937
Method:                 Least Squares   F-statistic:                     249.5
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           1.83e-46
Time:                        19:39:52   Log-Likelihood:                -51.764
No. Observations:                  84   AIC:                             115.5
Df Residuals:                      78   BIC:                             130.1
Df Model:                           5                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
-----------------------------------------------------------------------------------
const              -4.1755      4.075     -1.025      0.309     -12.288       3.937
bprice_relative     0.9063      0.026     34.805      0.000       0.854       0.958
wprice_relative     0.3572      0.138      2.580      0.012       0.082       0.633
tprice_relative     0.9063      0.026     34.805      0.000       0.854       0.958
q1                 -0.1644      0.144     -1.143      0.257      -0.451       0.122
q2                 -0.1672      0.145     -1.152      0.253      -0.456       0.122
q3                 -0.0001      0.144     -0.001      0.999      -0.287       0.287
==============================================================================
Omnibus:                        1.236   Durbin-Watson:                   0.609
Prob(Omnibus):                  0.539   Jarque-Bera (JB):                0.672
Skew:                           0.069   Prob(JB):                        0.714
Kurtosis:                       3.416   Cond. No.                     2.33e+17
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The smallest eigenvalue is 1.36e-30. This might indicate that there are
strong multicollinearity problems or that the design matrix is singular.

Second Stage Results (Demand Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.272
Model:                            OLS   Adj. R-squared:                  0.225
Method:                 Least Squares   F-statistic:                     5.833
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           0.000125
Time:                        19:39:52   Log-Likelihood:                 107.20
No. Observations:                  84   AIC:                            -202.4
Df Residuals:                      78   BIC:                            -187.8
Df Model:                           5                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
const                0.3679      0.335      1.099      0.275      -0.299       1.034
predicted_cprice    -0.0161      0.006     -2.815      0.006      -0.028      -0.005
tprice               0.0302      0.020      1.539      0.128      -0.009       0.069
q1                  -0.0900      0.022     -4.132      0.000      -0.133      -0.047
q2                  -0.0681      0.022     -3.133      0.002      -0.111      -0.025
q3                  -0.0823      0.022     -3.797      0.000      -0.125      -0.039
==============================================================================
Omnibus:                       29.401   Durbin-Watson:                   2.104
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               65.632
Skew:                           1.226   Prob(JB):                     5.60e-15
Kurtosis:                       6.569   Cond. No.                     1.02e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 1.02e+03. This might indicate that there are
strong multicollinearity or other numerical problems.


#Step 3: Run 2SLS for supply  BASIC not work
# 'cprice' is the endogenous variable for supply, predicted by IV
# Instruments (IV) for supply: 'log_income' 
# No control variables

# Prepare the independent variables for the first stage (instruments)
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
supply_iv = data[['log_income']]
supply_iv = sm.add_constant(supply_iv)  # Add constant

# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, supply_iv).fit()
predicted_cprice = first_stage_model.predict(supply_iv)

# Second Stage: Use predicted cprice to estimate the supply function
# Prepare supply function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
supply_X = pd.DataFrame({    
    'predicted_cprice': predicted_cprice})
supply_X = sm.add_constant(supply_X)  # Add constant term

# Assuming supply_y is the quantity supplied (you should use the correct column name here)
supply_y = data['qu']  # Replace 'qu' with the actual supply quantity column if necessary

# Second stage regression: Predict supply with predicted cprice
second_stage_model = sm.OLS(supply_y, supply_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Supply Function with Predicted cprice):")
print(second_stage_model.summary())
First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.266
Model:                            OLS   Adj. R-squared:                  0.257
Method:                 Least Squares   F-statistic:                     29.71
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           5.17e-07
Time:                        19:52:48   Log-Likelihood:                -157.76
No. Observations:                  84   AIC:                             319.5
Df Residuals:                      82   BIC:                             324.4
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const        -63.8192     14.064     -4.538      0.000     -91.797     -35.841
log_income    10.1189      1.857      5.450      0.000       6.426      13.812
==============================================================================
Omnibus:                       10.385   Durbin-Watson:                   0.117
Prob(Omnibus):                  0.006   Jarque-Bera (JB):               11.428
Skew:                           0.903   Prob(JB):                      0.00330
Kurtosis:                       2.941   Cond. No.                         620.
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.

Second Stage Results (Supply Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.010
Model:                            OLS   Adj. R-squared:                 -0.002
Method:                 Least Squares   F-statistic:                    0.8602
Date:                Tue, 17 Sep 2024   Prob (F-statistic):              0.356
Time:                        19:52:48   Log-Likelihood:                 94.301
No. Observations:                  84   AIC:                            -184.6
Df Residuals:                      82   BIC:                            -179.7
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
const                0.5729      0.117      4.880      0.000       0.339       0.807
predicted_cprice     0.0085      0.009      0.927      0.356      -0.010       0.027
==============================================================================
Omnibus:                       30.215   Durbin-Watson:                   1.633
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               73.193
Skew:                           1.220   Prob(JB):                     1.28e-16
Kurtosis:                       6.867   Cond. No.                         175.
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.

[8]
#Step 3: Run 2SLS for supply  OPTION 1: log income only: fail
# 'cprice' is the endogenous variable for supply, predicted by IV
# Instruments (IV) for supply: 'log_income' 
# Control variables: 'bprice''wprice'

# Prepare the independent variables for the first stage (instruments)
data['bprice_relative']=data['bprice']/data['oprice']
data['wprice_relative']=data['wprice']/data['oprice']
data['tprice_relative']=data['bprice']/data['oprice']
data['cprice_relative']=data['cprice']/data['oprice']
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
supply_iv = data[['log_income','bprice_relative','wprice_relative']]
supply_iv = sm.add_constant(supply_iv)  # Add constant

# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, supply_iv).fit()
predicted_cprice = first_stage_model.predict(supply_iv)

# Second Stage: Use predicted cprice to estimate the supply function
# Prepare supply function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
supply_X = pd.DataFrame({    
    'predicted_cprice': predicted_cprice,
    'bprice_relative':data['bprice_relative'],
    'wprice_relative':data['wprice_relative']})
supply_X = sm.add_constant(supply_X)  # Add constant term

# Assuming supply_y is the quantity supplied (you should use the correct column name here)
supply_y = data['qu']  # Replace 'qu' with the actual supply quantity column if necessary

# Second stage regression: Predict supply with predicted cprice
second_stage_model = sm.OLS(supply_y, supply_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Supply Function with Predicted cprice):")
print(second_stage_model.summary())
First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.948
Model:                            OLS   Adj. R-squared:                  0.946
Method:                 Least Squares   F-statistic:                     483.2
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           3.87e-51
Time:                        19:46:17   Log-Likelihood:                -46.820
No. Observations:                  84   AIC:                             101.6
Df Residuals:                      80   BIC:                             111.4
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
-----------------------------------------------------------------------------------
const             -12.8641      4.505     -2.856      0.005     -21.829      -3.899
log_income          2.3827      0.660      3.612      0.001       1.070       3.696
bprice_relative     1.6992      0.058     29.061      0.000       1.583       1.816
wprice_relative     0.0479      0.152      0.316      0.753      -0.254       0.350
==============================================================================
Omnibus:                        6.694   Durbin-Watson:                   0.609
Prob(Omnibus):                  0.035   Jarque-Bera (JB):                6.039
Skew:                           0.573   Prob(JB):                       0.0488
Kurtosis:                       3.640   Cond. No.                     2.91e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.91e+03. This might indicate that there are
strong multicollinearity or other numerical problems.

Second Stage Results (Supply Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.094
Model:                            OLS   Adj. R-squared:                  0.060
Method:                 Least Squares   F-statistic:                     2.773
Date:                Tue, 17 Sep 2024   Prob (F-statistic):             0.0468
Time:                        19:46:17   Log-Likelihood:                 98.018
No. Observations:                  84   AIC:                            -188.0
Df Residuals:                      80   BIC:                            -178.3
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
const                0.6846      0.698      0.980      0.330      -0.705       2.074
predicted_cprice     0.0969      0.049      1.962      0.053      -0.001       0.195
bprice_relative     -0.1919      0.090     -2.129      0.036      -0.371      -0.013
wprice_relative     -0.0185      0.028     -0.652      0.517      -0.075       0.038
==============================================================================
Omnibus:                       34.565   Durbin-Watson:                   1.777
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              101.599
Skew:                           1.318   Prob(JB):                     8.67e-23
Kurtosis:                       7.699   Cond. No.                     2.67e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.67e+03. This might indicate that there are
strong multicollinearity or other numerical problems.


#Step 3: Run 2SLS for supply  OPTION 2: log bprice, wprice and income
# 'cprice' is the endogenous variable for supply, predicted by IV
# Instruments (IV) for supply: 'log_income' 
# Control variables: 'bprice''wprice'

# Prepare the independent variables for the first stage (instruments)
data['log_bprice_relative']=np.log(data['bprice']/data['oprice'])
data['log_wprice_relative']=np.log(data['wprice']/data['oprice'])
data['log_income'] = np.log(data['income'])
cprice_relative = data['cprice_relative']
supply_iv = data[['log_income','log_bprice_relative','log_wprice_relative']]
supply_iv = sm.add_constant(supply_iv)  # Add constant

# First stage regression: Predict cprice
first_stage_model = sm.OLS(cprice_relative, supply_iv).fit()
predicted_cprice = first_stage_model.predict(supply_iv)

# Second Stage: Use predicted cprice to estimate the supply function
# Prepare supply function independent variables (including control variables)
# Use the predicted cprice instead of the original cprice
supply_X = pd.DataFrame({    
    'predicted_cprice': predicted_cprice,
    'log_bprice_relative':data['log_bprice_relative'],
    'log_wprice_relative':data['log_wprice_relative']})
supply_X = sm.add_constant(supply_X)  # Add constant term

# Assuming supply_y is the quantity supplied (you should use the correct column name here)
supply_y = data['qu']  # Replace 'qu' with the actual supply quantity column if necessary

# Second stage regression: Predict supply with predicted cprice
second_stage_model = sm.OLS(supply_y, supply_X).fit()

# Print the results
print("First Stage Results (Predicting cprice using IVs):")
print(first_stage_model.summary())

print("\nSecond Stage Results (Supply Function with Predicted cprice):")
print(second_stage_model.summary())



First Stage Results (Predicting cprice using IVs):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:        cprice_relative   R-squared:                       0.913
Model:                            OLS   Adj. R-squared:                  0.910
Method:                 Least Squares   F-statistic:                     280.3
Date:                Tue, 17 Sep 2024   Prob (F-statistic):           2.48e-42
Time:                        19:50:20   Log-Likelihood:                -68.131
No. Observations:                  84   AIC:                             144.3
Df Residuals:                      80   BIC:                             154.0
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
=======================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------
const                 -47.8527     17.292     -2.767      0.007     -82.264     -13.441
log_income              1.9929      0.871      2.288      0.025       0.260       3.726
log_bprice_relative     6.7520      0.309     21.851      0.000       6.137       7.367
log_wprice_relative    10.9734      5.953      1.843      0.069      -0.873      22.820
==============================================================================
Omnibus:                        6.617   Durbin-Watson:                   0.406
Prob(Omnibus):                  0.037   Jarque-Bera (JB):                6.268
Skew:                           0.663   Prob(JB):                       0.0435
Kurtosis:                       3.180   Cond. No.                     2.53e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.53e+03. This might indicate that there are
strong multicollinearity or other numerical problems.

Second Stage Results (Supply Function with Predicted cprice):
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                     qu   R-squared:                       0.100
Model:                            OLS   Adj. R-squared:                  0.066
Method:                 Least Squares   F-statistic:                     2.964
Date:                Tue, 17 Sep 2024   Prob (F-statistic):             0.0370
Time:                        19:50:20   Log-Likelihood:                 98.289
No. Observations:                  84   AIC:                            -188.6
Df Residuals:                      80   BIC:                            -178.9
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
=======================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------
const                   7.0512      4.262      1.655      0.102      -1.430      15.532
predicted_cprice        0.1252      0.060      2.078      0.041       0.005       0.245
log_bprice_relative    -0.9616      0.433     -2.219      0.029      -1.824      -0.099
log_wprice_relative    -2.0027      1.318     -1.519      0.133      -4.626       0.621
==============================================================================
Omnibus:                       35.557   Durbin-Watson:                   1.774
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              106.998
Skew:                           1.351   Prob(JB):                     5.83e-24
Kurtosis:                       7.824   Cond. No.                     7.18e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 7.18e+03. This might indicate that there are
strong multicollinearity or other numerical problems.


