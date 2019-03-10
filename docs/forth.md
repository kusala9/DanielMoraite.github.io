![Machine Learning](/images/robo28.jpg)

# Machine Learning

I. Looking at some old and new algorithms and ensembles
   - Warming up with a XGBoost 

II. Interesting applications and solutions 

III. Some of my favorite projects


------------------
I. ### XGBoost

   Now let's have some fun with the previous data sets that we've clean with pandas.

   Since XGBoost is so popular, as an ensemble of decision trees, I thought to have a look at how it works on simple data sets (Titanic/survivors and Pima/diabets). I'll just have a very quick data cleaning and then will be scoring the XGBoost models.

First set: classification: 

    import pandas as pd
    from sklearn.model_selection import train_test_split
    from xgboost import XGBClassifier

    data = pd.read_csv("titanic.csv")

    # just a little bit of cleaning up:
    data['Sex'] = data['Sex'].map({'male': 0, 'female': 1})
    data = data.dropna()
    
    X = data.drop('Survived', axis=1)
    y = data['Survived']

Now ready to split the data for training, chose the model and fiting it:

    X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=1)
    
    model = XGBClassifier()
    model.fit(X_train, y_train)

    y_pred = model.predict(X_test)
    predictions = [round(value) for value in y_pred]

And look at the prediction model accuracy: which is 82.68%

    from sklearn.metrics import accuracy_score
    accuracy = accuracy_score(y_test, predictions)
    print("Accuracy: %.2f%%" % (accuracy * 100.0))

The other set - regression: 

    from numpy import loadtxt 

    dataset = loadtxt('pima.txt', delimiter=",")

    X = dataset[:,0:8] 
    Y = dataset[:,8]

    seed = 7 #reproducibility of your model
    test_size = 0.30 # 30% for testing

    X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=test_size, random_state=seed)

    model = XGBClassifier() 
    model.fit(X_train, y_train)

Early stopping is used to avoid overfitting - since gradient boost models are prone for it.

     eval_set = [(X_test, y_test)] 
     model.fit(X_train, y_train, eval_metric="error", eval_set=eval_set, verbose=True)
     # note that the parameter has been added. 

    [30]	validation_0-error:0.207792
    [31]	validation_0-error:0.207792
    [32]	validation_0-error:0.212121
    [33]	validation_0-error:0.207792
    [34]	validation_0-error:0.207792
    [35]	validation_0-error:0.21645
    [36]	validation_0-error:0.207792
    [37]	validation_0-error:0.207792
    [38]	validation_0-error:0.21645
    [39]	validation_0-error:0.212121
    # The parameter is early_stopping_rounds=36

    eval_set = [(X_test, y_test)] 
    model.fit(X_train, y_train, eval_metric="error",early_stopping_rounds=36, eval_set=eval_set, verbose=True)
    # Will train until validation_0-error hasn't improved in 36 rounds
    .....
    [64]	validation_0-error:0.220779
    [65]	validation_0-error:0.21645
    [66]	validation_0-error:0.21645
    Stopping. Best iteration:
    [30]	validation_0-error:0.207792

    y_pred = model.predict(X_test) 
    predictions = [round(value) for value in y_pred] 
    accuracy = accuracy_score(y_test, predictions) 
    print("Accuracy: %.2f%%" % (accuracy * 100.0))
    Accuracy: 79.22%

Now let's play with Kfold:

    from sklearn.model_selection import StratifiedKFold 
    from sklearn.model_selection import cross_val_score
    
    model = XGBClassifier()
    kfold = StratifiedKFold(n_splits=10, random_state=7) 
    results = cross_val_score(model, X, Y, cv=kfold)

    print("Accuracy: %.2f%% (%.2f%%)" % (results.mean()*100, results.std()*100))
    Accuracy: 76.95% (5.88%)

To see the rest of iterations please have a look at the jupyter notebook [here](https://github.com/DanielMoraite/DanielMoraite.github.io/blob/master/assets/XGBoost.ipynb).

-------------------

### Coming soon:

II. Interesting applications and solutions 

III. Some of my favorite projects

----------------
----------------
