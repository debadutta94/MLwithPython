# MLwithPython

#Classification with Python
import itertools
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.ticker import NullFormatter
import pandas as pd
import numpy as np
import matplotlib.ticker as ticker
from sklearn import preprocessing
%matplotlib inline
!wget -O loan_train.csv https://s3-api.us-geo.objectstorage.softlayer.net/cf-courses-data/CognitiveClass/ML0101ENv3/labs/loan_train.csv
df = pd.read_csv('loan_train.csv')
df.head()
df.shape
df['due_date'] = pd.to_datetime(df['due_date'])
df['effective_date'] = pd.to_datetime(df['effective_date'])
df.head()

#Data visualization and pre-processing
import seaborn as sns

bins = np.linspace(df.Principal.min(), df.Principal.max(), 10)
g = sns.FacetGrid(df, col="Gender", hue="loan_status", palette="Set1", col_wrap=2)
g.map(plt.hist, 'Principal', bins=bins, ec="k")

g.axes[-1].legend()
plt.show()
bins = np.linspace(df.age.min(), df.age.max(), 10)
g = sns.FacetGrid(df, col="Gender", hue="loan_status", palette="Set1", col_wrap=2)
g.map(plt.hist, 'age', bins=bins, ec="k")

g.axes[-1].legend()
plt.show()

df['dayofweek'] = df['effective_date'].dt.dayofweek
bins = np.linspace(df.dayofweek.min(), df.dayofweek.max(), 10)
g = sns.FacetGrid(df, col="Gender", hue="loan_status", palette="Set1", col_wrap=2)
g.map(plt.hist, 'dayofweek', bins=bins, ec="k")
g.axes[-1].legend()
plt.show()
df['weekend'] = df['dayofweek'].apply(lambda x: 1 if (x>3)  else 0)
df.head()


df['weekend'].value_counts()
df['dayofweek'].value_counts()
df.groupby(['Gender'])['loan_status'].value_counts(normalize=True)
df['Gender'].replace(to_replace=['male','female'], value=[0,1],inplace=True)
df.head()


df.groupby(['education'])['loan_status'].value_counts(normalize=True)
df[['Principal','terms','age','Gender','education']].head()
Feature = df[['Principal','terms','age','Gender','weekend']]
Feature = pd.concat([Feature,pd.get_dummies(df['education'])], axis=1)
Feature.drop(['Master or Above'], axis = 1,inplace=True)
Feature.head()

X = Feature
X[0:5]
y = df['loan_status'].values
y[0:5]
X= preprocessing.StandardScaler().fit(X).transform(X)
X[0:5]





#K Nearest Neighbor(KNN)
#Decision Tree
#Support Vector Machine
#Logistic Regression
from sklearn.model_selection import train_test_split
from sklearn import metrics
from sklearn.neighbors import KNeighborsClassifier
X_train, X_test, y_train, y_test = train_test_split( X, y, test_size=0.2, random_state=4)
print ('Train set:', X_train.shape,  y_train.shape)
print ('Test set:', X_test.shape,  y_test.shape)

Ks = 10
mean_acc = np.zeros((Ks-1))
std_acc = np.zeros((Ks-1))

for n in range(1,Ks):
    
    #Train Model and Predict  
    neigh = KNeighborsClassifier(n_neighbors = n).fit(X_train,y_train)
    yhat=neigh.predict(X_test)
    mean_acc[n-1] = metrics.accuracy_score(y_test, yhat)

    
    std_acc[n-1]=np.std(yhat==y_test)/np.sqrt(yhat.shape[0])

mean_acc
kmax = mean_acc.argmax()+1
print( "The best accuracy was with", mean_acc.max(), "with k=", kmax) 


from sklearn.tree import DecisionTreeClassifier
loanTree = DecisionTreeClassifier(criterion="entropy", max_depth = 4)
loanTree.fit(X,y)

#Support Vector Machine
from sklearn import svm
from sklearn.metrics import f1_score
from sklearn.metrics import jaccard_score
kernels = ['rbf', 'linear', 'poly', 'sigmoid']
clf = []
svm_acc_jac = np.zeros(4)
svm_acc_f1 = np.zeros(4)

for ker in range(3):
    clf.append(svm.SVC(kernel=kernels[ker]))
    clf[ker].fit(X_train, y_train)
    yhat = clf[ker].predict(X_test)
    svm_acc_jac[ker] = f1_score(y_test, yhat, average='weighted')
    svm_acc_f1[ker] = jaccard_score(y_test, yhat,pos_label='PAIDOFF')

bestKer_Jac = kernels[svm_acc_jac.argmax()]
bestKer_F1 = kernels[svm_acc_f1.argmax()]

print("The best jaccard accuracy was ", svm_acc_jac.max(), "with kernal=", bestKer_Jac) 
print ("The best F1_score accuracy was ", svm_acc_f1.max(), "with kernal=", bestKer_F1)
from sklearn.linear_model import LogisticRegression
LR = LogisticRegression(C=0.01, solver='liblinear').fit(X,y)
LR

#Model Evaluation using Test set
#from sklearn.metrics import jaccard_similarity_score
from sklearn.metrics import jaccard_score
from sklearn.metrics import f1_score
from sklearn.metrics import log_loss
!wget -O loan_test.csv https://s3-api.us-geo.objectstorage.softlayer.net/cf-courses-data/CognitiveClass/ML0101ENv3/labs/loan_test.csv
test_df = pd.read_csv('loan_test.csv')
test_df.head()
# Pre-processing
test_df['due_date'] = pd.to_datetime(test_df['due_date'])
test_df['effective_date'] = pd.to_datetime(test_df['effective_date'])

test_df['dayofweek'] = test_df['effective_date'].dt.dayofweek 
test_df['weekend'] = test_df['dayofweek'].apply(lambda x: 1 if (x>3)  else 0)

#test_df['Gender'].replace(to_replace=['male','female'], value=[0,1],inplace=True)
test_df['Gender']=test_df['Gender'].replace({'male':0,'female':1})

test_Feature = test_df[['Principal','terms','age','Gender','weekend']]
test_Feature = pd.concat([test_Feature,pd.get_dummies(test_df['education'])], axis=1)
test_Feature.drop(['Master or Above'], axis = 1,inplace=True)
test_Feature.head()
test_Feature.shape
X_test = test_Feature
y_test = test_df['loan_status'].values

# K-nearest neighbors

neigh = KNeighborsClassifier(n_neighbors = kmax).fit(X,y)
yhat=neigh.predict(X_test)
knn_row = ['KNN', jaccard_score(y_test, yhat, average='weighted'), f1_score(y_test, yhat, pos_label='PAIDOFF'), 'NA']
print(knn_row)

# Decision Tree

predTree = loanTree.predict(X_test)
dt_row = ['Decision Tree', jaccard_score(y_test, predTree, average='weighted'), f1_score(y_test, predTree, pos_label='PAIDOFF'), 'NA']
print(dt_row)


# Support Vector Machine

clf_rbf= svm.SVC(kernel=bestKer_Jac)
clf_rbf.fit(X, y)
yhat_rbf = clf_rbf.predict(X_test)
svm_acc_jac = f1_score(y_test, yhat_rbf, average='weighted')

clf_li= svm.SVC(kernel=bestKer_F1)
clf_li.fit(X, y)
yhat_li = clf_li.predict(X_test)
svm_acc_f1 = jaccard_score(y_test, yhat_li,pos_label='PAIDOFF')

svm_row = ['SVM', svm_acc_jac, svm_acc_f1, 'NA']
print(svm_row)

# Logistic Regression

yhat = LR.predict(X_test)
yhat_prob = LR.predict_proba(X_test)
lr_row = ['Logistic Regression', jaccard_score(y_test, yhat,pos_label='PAIDOFF'), f1_score(y_test, yhat, average='weighted'),log_loss(y_test, yhat_prob)]
print(lr_row)
result_df = pd.DataFrame(data = [knn_row,dt_row, svm_row,lr_row], columns = ['Algorithm', 'Jaccard', 'F1-score', 'Logloss'])
print(result_df)
