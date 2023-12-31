﻿# Lead_score_Data
import warnings
warnings.filterwarnings('ignore')

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import datetime

from sklearn.preprocessing import scale
from sklearn.preprocessing import StandardScaler

from sklearn.datasets import fetch_openml
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn import metrics
from sklearn.model_selection import train_test_split
import statsmodels.api as sm
from sklearn.metrics import precision_score, recall_score
from sklearn.metrics import precision_recall_curve
from random import sample
from numpy.random import uniform
import numpy as np
from math import isnan

pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 200)

Lead_score= pd.read_csv ('Leads.csv')
Lead_score_backup= Lead_score

Lead_score.head()

Lead_score.shape

Lead_score.info()

Lead_score.describe()

# Checking for duplicate rows in the dataset
Lead_score.loc[Lead_score.duplicated()]

Lead_score = Lead_score.replace('Select', np.nan)

# % of null values
round(100*(Lead_score.isnull().sum()/len(Lead_score.index)), 2)

null_percentage = Lead_score.isnull().mean() * 100

columns_to_drop = null_percentage[null_percentage > 70].index
Lead_score = Lead_score.drop(columns_to_drop, axis=1)

round(100*(Lead_score.isnull().sum()/len(Lead_score.index)), 2)

Lead_score['Country'] = Lead_score['Country'].apply(lambda x: 'India' if x=='India' else 'Outside India')
Lead_score['Country'].value_counts()

Lead_score['Country'].isnull().sum()

Lead_score.loc[pd.isnull(Lead_score['Country']), ['Country']] = 'India'

Lead_score['Country'].isnull().sum()

Lead_score['Specialization'].value_counts()

# Lets check the number of null values in this column
Lead_score['Specialization'].isnull().sum()

# Since the amount of null values is high , Lets compute the null values and replace them with text 'Unknown'
Lead_score['Specialization'].fillna("Unknown", inplace = True)
Lead_score['Specialization'].value_counts()

Lead_score['What is your current occupation'].isnull().sum()

Lead_score['What is your current occupation'].value_counts()

Lead_score['What is your current occupation'].fillna("Unknown", inplace = True)
Lead_score['What is your current occupation'].value_counts()

Lead_score['What matters most to you in choosing a course'].isnull().sum()

Lead_score['What matters most to you in choosing a course'].unique()

# Since this column does not give away a lot of information and mostly are null or 'other' we can drop the column
Lead_score = Lead_score.drop('What matters most to you in choosing a course' , axis= 1)

Lead_score['Tags'].value_counts()

# This column does not add much value to the analysis , hence dropping it. 
Lead_score = Lead_score.drop('Tags',axis =1)

Lead_score['Lead Quality'].unique()

sns.countplot(data =Lead_score, x='Lead Quality')
plt.show()

# Here all the null values are as good as not sure.

Lead_score['Lead Quality'] = Lead_score['Lead Quality'].replace(np.nan, 'Not Sure')

sns.countplot(data=Lead_score,x= 'Lead Quality')
    

Lead_score['City'].fillna("unknown",inplace = True)
Lead_score['City'].value_counts()

Lead_score = Lead_score.drop(['Asymmetrique Activity Score', 'Asymmetrique Profile Score'], axis=1)

Lead_score['Asymmetrique Activity Index'].fillna("Unknown", inplace = True)
Lead_score['Asymmetrique Activity Index'].value_counts()
Lead_score['Asymmetrique Profile Index'].fillna("Unknown", inplace = True)
Lead_score['Asymmetrique Profile Index'].value_counts()

print(Lead_score.columns)

print(Lead_score['Magazine'].value_counts())
print(Lead_score['Receive More Updates About Our Courses'].value_counts())
print(Lead_score['Update me on Supply Chain Content'].value_counts())
print(Lead_score['I agree to pay the amount through cheque'].value_counts())

Lead_score= Lead_score.loc[:,Lead_score.nunique()!=1]


# Prospect ID gives out the same information as Lead Number , therefore we can drop it from  the dataset 
Lead_score = Lead_score.drop('Prospect ID' , axis=1)

print(Lead_score.columns)


round(100*(Lead_score.isnull().sum()/len(Lead_score.index)), 2)

print(Lead_score['Do Not Email'].value_counts())
print(Lead_score['Do Not Call'].value_counts())
print(Lead_score['Search'].value_counts())
print(Lead_score['Through Recommendations'].value_counts())
print(Lead_score['A free copy of Mastering The Interview'].value_counts())
print(Lead_score['Newspaper Article'].value_counts())
print(Lead_score['X Education Forums'].value_counts())
print(Lead_score['Newspaper'].value_counts())
print(Lead_score['Digital Advertisement'].value_counts())

# Since these columns do not give away any specific information we can drop these columns .
Lead_score = Lead_score.drop(columns =['Do Not Email' , 'Do Not Call' , 'Search' ,'Through Recommendations' ,'A free copy of Mastering The Interview','Newspaper Article', 'X Education Forums', 'Newspaper', 
            'Digital Advertisement'],axis=1)

Lead_score.columns

Lead_score.shape

# Lets drop rows with null values
Lead_score= Lead_score.dropna()

round(100*(Lead_score.isnull().sum()/len(Lead_score.index)), 2)

# Creating dummy variables for categorial variables

categorical_columns = ['Lead Origin' ,'Country','Lead Quality' ,'Lead Source','Last Activity' , 'Specialization' , 'What is your current occupation'
                                       ,'City' ,'Last Notable Activity' ,'Asymmetrique Activity Index' , 'Asymmetrique Profile Index']

for x in categorical_columns:
    cont = pd.get_dummies(Lead_score[x],prefix=x,drop_first=True)
    Lead_score = pd.concat([Lead_score,cont],axis=1)

Lead_score.shape

print(Lead_score.columns)

Lead_score.head(1)

#dropping the original columns we created dummies for 
Lead_score = Lead_score.drop(columns = ['Lead Origin' ,'Country','Lead Quality' ,'Lead Source','Last Activity' , 'Specialization' , 'What is your current occupation'
                                       ,'City' ,'Last Notable Activity' ,'Asymmetrique Activity Index' , 'Asymmetrique Profile Index'] , axis =1)

Lead_score.shape

Lead_score.head()

#checking for outliers 
outlier_check = Lead_score[['TotalVisits','Total Time Spent on Website','Page Views Per Visit']]

outlier_check.describe(percentiles=[.01,.1,.2,.25, .5, .75, .90, .95, .99])

Q1 = Lead_score['Page Views Per Visit'].quantile(0.25)
Q3 = Lead_score['Page Views Per Visit'].quantile(0.75)
IQR = Q3 - Q1
Lead_score=Lead_score.loc[(Lead_score['Page Views Per Visit'] >= Q1 - 1.5*IQR) & (Lead_score['Page Views Per Visit'] <= Q3 + 1.5*IQR)]

Q1 = Lead_score['TotalVisits'].quantile(0.25)
Q3 = Lead_score['TotalVisits'].quantile(0.75)
IQR = Q3 - Q1
Lead_score=Lead_score.loc[(Lead_score['TotalVisits'] >= Q1 - 1.5*IQR) & (Lead_score['TotalVisits'] <= Q3 + 1.5*IQR)]

Q1 = Lead_score['Total Time Spent on Website'].quantile(0.25)
Q3 = Lead_score['Total Time Spent on Website'].quantile(0.75)
IQR = Q3 - Q1
Lead_score=Lead_score.loc[(Lead_score['Total Time Spent on Website'] >= Q1 - 1.5*IQR) & (Lead_score['Total Time Spent on Website'] <= Q3 + 1.5*IQR)]

#Test Train SPlit
X = Lead_score.drop(['Lead Number','Converted'], axis=1)

X.head()

y = Lead_score['Converted']

y.head()

# Splitting the data into train and test
X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.7, test_size=0.3, random_state=100)

#scaling continuous variables
scaler = StandardScaler()

X_train[['TotalVisits','Total Time Spent on Website','Page Views Per Visit']] = scaler.fit_transform(X_train[['TotalVisits','Total Time Spent on Website','Page Views Per Visit']])

X_train.head()

converted = (sum(Lead_score['Converted'])/len(Lead_score['Converted'].index))*100
converted

Lead_score.head()

for column in Lead_score.columns:
    if Lead_score[column].dtype == bool:
        Lead_score[column] = Lead_score[column].astype(int)

Lead_score.head()

print(Lead_score.dtypes)

for column in Lead_score.columns:
    if Lead_score[column].dtype == float:
        Lead_score[column] = Lead_score[column].astype(int)

Lead_score.head()

#Logistic regression model
logistics = sm.GLM(y_train,(sm.add_constant(X_train)), family = sm.families.Binomial())
logistics.fit().summary()

print(Lead_score.colums.dtypes)

print(Lead_score.dtypes)

Lead_score.head()

num_columns = Lead_score.shape[1]
print(num_columns)

Lead_score.shape

Lead_score.head()

X = df.drop('Lead Number', axis=1)
y = df['Lead Number']

X = sm.add_constant(X)

logistics = sm.GLM(y, X, family=sm.families.Binomial())

logistics_results = logistics.fit()

print(logistics_results)

Lead_score.head()

logistics.fit().summary()

X = Lead_score.drop('Converted', axis=1)
y = Lead_score['Converted']

X = sm.add_constant(X)

logistics = sm.GLM(y, X, family=sm.families.Binomial())

logistics_results = logistics.fit()

logistics.fit().summary()

from sklearn.linear_model import LogisticRegression
logreg = LogisticRegression()

  # running RFE with 20 variables as output
from sklearn.feature_selection import RFE         

from sklearn.linear_model import LogisticRegression

estimator = LogisticRegression() 
num_features = 20

# Create a logistic regression model
logistic_model = LogisticRegression()

# Create RFE object with 20 features to select
rfe = RFE(estimator=logistic_model, n_features_to_select=20)

# Fit RFE to the data
rfe= rfe.fit(X_train, y_train)

# Get the selected features
rfe.support_

list(zip(X_train.columns, rfe.support_, rfe.ranking_))

X_train.columns[~rfe.support_]

cols = X_train.columns[rfe.support_]

X_train_sm = sm.add_constant(X_train[cols])

X = Lead_score.drop('Converted', axis=1)
y = Lead_score['Converted']

X_train_sm = sm.add_constant(X_train[cols])

X = sm.add_constant(X)

logistics = sm.GLM(y, X, family=sm.families.Binomial())

log = logistics.fit()

log.summary()

vif = pd.DataFrame()

vif['Features'] = X_train[cols].columns

vif['VIF'] = [vif(X_train[cols].values, i) for i in range(X_train[cols].shape[1])]

