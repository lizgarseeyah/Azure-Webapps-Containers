## Project Overview

### Resources Used:

- Jupyter Notebook 6.1.3 
- Python 3.6

### File Descriptions:

- [finding_donors.ipynb](https://github.com/lizgarseeyah/Finding-Donors/blob/master/finding_donors.ipynb) - main file
-  [p1_charityml.zip](https://github.com/lizgarseeyah/Finding-Donors/blob/master/p1_charityml_rev2.zip) - zip folder containing csv data sources for the program.

### Summary:

This project applies and evaluates three types of supervised learning models, Ensemble, K-NN, and SVM, to identify potential donors. Each model is evaluated and scored for accuracy and measured on f-score.

## Problem Statement:

CharityML is a fictitious charity organization located in the heart of Silicon Valley that was established to provide financial support for people eager to learn machine learning. After nearly 32,000 letters were sent to people in the community, CharityML determined that every donation they received came from someone that was making more than $50,000 annually. To expand their potential donor base, CharityML has decided to send letters to residents of California, but to only those most likely to donate to the charity. 

With nearly 15 million working Californians, CharityML has brought you on board to help build an algorithm to best identify potential donors and reduce overhead cost of sending mail. Your goal will be evaluate and optimize several different supervised learners to determine which algorithm will provide the highest donation yield while also reducing the total number of letters being sent.

## Solution: 

The procedure below is a high-level summary of the steps I took to address the problem statement. For a more detailed explaination, please see the [finding_donors.ipynb](https://github.com/lizgarseeyah/Finding-Donors/blob/master/finding_donors.ipynb) file in the GitHub repository.

### Data Exploration and preprocessing

The first part of this program imports and preprocesses the data. Before preprocessing, one must select a feature set that addresses the problem statment. The next step normalizes the data by handling missing, invalid, or outlying data points by either removing or performing a method called **one-hot encoding**. The table below shows how one-hot encoding works: the function takes a categorical data type and changes to a numerical data so that the ML algorithm can process the data.


![one-hot-encoding](/img/One-Hot-encoding.png) 

### Train and test the data using a supervised learning method 

The first step before using the training models is to shuffle and split the data into a training set (80% of the data) and a testing set (20% of the data).

```markdown
# Import train_test_split
from sklearn.cross_validation import train_test_split

# Split the 'features' and 'income' data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(features_final, 
                                                    income, 
                                                    test_size = 0.2, 
                                                    random_state = 0)

# Show the results of the split
print("Training set has {} samples.".format(X_train.shape[0]))
print("Testing set has {} samples.".format(X_test.shape[0]))
```
The training set will be used to train the data on three supervised learning models: AdaBoost Ensemble, K-Nearest Neighbors (KNN), and Support Vecotr Machine (SVM).

```markdown
# Import the three supervised learning models from sklearn
from sklearn.ensemble import AdaBoostClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier

## Initialize the three models
clf_A = SVC(random_state=1)#SVC
clf_B = KNeighborsClassifier()
clf_C = AdaBoostClassifier(random_state=1)#Ensemble Methods
```
An initial model evaluation was performed by running 1%, 10%, and 100% of the training data through each model. The accuracy and f-score metrics are used to determine the best model. Accuracy is useful here, because we do not want to request donations from a person that we incorrectly identified as a potential donor. The f-score measures both precision, true positives, and the training models' ability to recall those individuals.

![accuracy-f-score](/img/accuracy-f-score.png) 
![performance-metrics](/img/performance-metrics.png) 

After completing the evaluation, the testing set results above indicated that the AdaBoost Classifier was the best model for this project based on time, accuracy score, and f-score.

**AdaBoost Overview**

Adaboost belongs to the family of ensemble methods. Ensemble means we take into account a set of multiple "weak" hypothesis and combine them to form one "strong" hypothesis. At each iteration, a "weak" hypothesis attempts to classify the training data, here for example it tries to approximately find the individuals making more than fifty-thousand dollars All the misclassified individuals are more heavily weighted and more focus will be given to the hard-to-classify points in the attempt to classify them correctly at the next iteration. Iteration after iteration, the combination of all those "weak" learners should converge towards a more confident, stronger hypothesis allowing to find individuals making more than fifty-thousand dollars. 

The only requirement for this model to converge well is that every "weak" learner need to be slightly better than random guessing. An analogy to this method would be to ask a crowd of unexperienced doctors to diagnose a disease rather than asking only one expert. The expert would more often be right than each doctor taken individually, but when considering the crowd, all the answers should converge towards the right outcome with more and more confidence as we ask more doctors (given that each unexperienced doctor is still doing better than a randomly guessing, that the previous doctor communicates the results of his analysis to the next one and that each doctor focuses on what the previous one wasn't able to find).
[src](https://www.analyticsvidhya.com/blog/2015/11/quick-introduction-boosting-algorithms-machine-learning/)

**Model Tuning**

To further improve the accuracy and f-score, a grid search is applied. Grid search brute forces all possible combinations of hyperparamters and selects the best inputs to be inserted into the AdaBoost model. 

```markdown
from sklearn.grid_search import GridSearchCV
from sklearn.metrics import make_scorer

# Initialize the classifier
clf = clf_C

# Create the parameters list you wish to tune, using a dictionary if needed.
# parameters = {'parameter_1': [value1, value2], 'parameter_2': [value1, value2]}
#parameters: base_estimator, n_estimator, learning_rate, random_state, algorithm
parameters = {'n_estimators': [25, 50, 75, 100], 'learning_rate': [0.25, 0.5, 0.75,1.0]}

# Make an fbeta_score scoring object using make_scorer()
scorer = make_scorer(fbeta_score, beta=0.5)

# Perform grid search on the classifier using 'scorer' as the scoring method using GridSearchCV()
grid_obj = GridSearchCV(clf,parameters, scorer)

# Fit the grid search object to the training data and find the optimal parameters using fit()
grid_fit = grid_obj.fit(X_train,y_train)

# Get the estimator
best_clf = grid_fit.best_estimator_

# Make predictions using the unoptimized and model
predictions = (clf.fit(X_train, y_train)).predict(X_test)
best_predictions = best_clf.predict(X_test)

# Report the before-and-afterscores
print("Unoptimized model\n------")
print("Accuracy score on testing data: {:.4f}".format(accuracy_score(y_test, predictions)))
print("F-score on testing data: {:.4f}".format(fbeta_score(y_test, predictions, beta = 0.5)))
print("\nOptimized Model\n------")
print("Final accuracy score on the testing data: {:.4f}".format(accuracy_score(y_test, best_predictions)))
print("Final F-score on the testing data: {:.4f}".format(fbeta_score(y_test, best_predictions, beta = 0.5)))
```
**Final Results**

![model-evaluation](/img/model-evaluation.png)

### Extracting Feature Importance

The final step in this project is determine which feature(s) have the most influence on the AdaBoost model. Features that I believe are relevant in predicting likely donors, ranging from most important to least are:

- occupation
- workclass
- relationship
- education level
- capital gain

It helps to have good subject matter on this, but my reasoning is that someone with good income, a good job, good education, not a lot of dependents, and capital gain are likely to give to charities.

```markdown
# Import a supervised learning model that has 'feature_importances_'
#from sklearn.ensemble import AdaBoostClassifier
from sklearn.ensemble import RandomForestClassifier
# Train the supervised model on the training set using .fit(X_train, y_train)
model = RandomForestClassifier().fit(X_train, y_train)

# Extract the feature importances using .feature_importances_ 
importances = model.feature_importances_

# Plot
vs.feature_plot(importances, X_train, y_train)
```
![feature-importance](/img/feature-importance.png) 

From the chart above, it shows three out of five selected features had the most influence on the AdaBoost model, these features are: education, relationship, and capital gain. In my opinion, I am surprised that occupation is not at least one of the relevant features. Feature importance is measured by looking at how much the scores decreases when a feature is not there. Which leads me to think that someone that makes more than fifty-thousand dollars is dependent on the level of education, whether they are married or not, the number of hours per week they work (stability), capital gain, and age. Race, sex, workclass, and education won't matter as much since this data isn't very descriptive and is varied(i.e. it is independent of a person's marital and financial status and won't contribute to the model and help predict whether a person makes more than $50K).

## Final Accurancy and F-score results

By modifying my model to take in the three featues, the final results are shown below:

![fs-final](/img/fs-final.png) 

With a limited feature set, the accuracy and f-score has decreased by 0.0137 (1.60%) and 0.0292 (4.16%), respectively. If training time was a factor, I would consider using a reduced data training set since the amount the scores decreased with the reduced training set is small.
