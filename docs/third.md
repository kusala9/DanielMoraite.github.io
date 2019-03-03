![robo](/images/robo22.jpeg)

# Pandas 

## Basics 

[Pandas](https://pandas.pydata.org/) on top of the cute name is one of the basic tools for data science. But first don't forget to install it: `conda install pandas`.

I won't spend much time here to let you know what is good for(read the data, perform analisys, output it neatly and plotting)  and I will just jump into a few data sets examples. 

### Examples: 

Of course there are somany different types of data and for each of them there is a better tool to use. I will just browse through some tools and practices that I came across several times by now. 

* Avocado set used for prediction of avocado prices:
You can warm up with this data set. Just had fun with it last weekend when received it from Bogdan: [Introduction - Data Analysis and Data Science with Python and Pandas](https://youtu.be/nLw1RNvfElg)

Easy to plot a simple plot with dates and prices:
![pandas](/images/pandas1.png)

Also fun to get your data to normalize your data: 

![pandas](/images/datanormalizer.png)

> of course there are other libraries that have preseted such functions, so you don't have to think about writting a formula.

* Pima predictions set used for predicting diabetes:
Here you can see a different data set where pandas combine so beautifully with numpy and matplotlib: I find correlation matrixes very apealing. 




Correlation Matrix 1 | Correlation Matrix 2
-------------------  | --------------------
![pandas](/images/correlmatrix2.png) | ![pandas](/images/correlmatrix3.png)

`def plot_corr(df, size=11):
          corr = df.corr()
          fig, ax = plt.subplots(figsize=(size, size))
          ax.matshow(corr)
          plt.xticks(range(len(corr.columns)), corr.columns)
          plt.yticks(range(len(corr.columns)), corr.columns)`


`rs = np.random.RandomState(0)
           corr = df.corr()
           corr.style.background_gradient()`
           

You can easily see in the second graph the correlation between the two columns(age and thickness), though not all the data sets are that friendly and easy to spot. 


> coming soon ... 

I will return with more examples and feedback (since I am planning to learn more about the full cycle of ... from data cleaning and using it for ML .. to API's) ... 

-----------------------------------

#### Tools and Resources: 

Practice makes perfect, and is always better to try your knowledge on data sets that you haven't worked during online classes. A best place to find all sorts of data is [Kaggle](https://www.kaggle.com/).

-------------------
-------------------



