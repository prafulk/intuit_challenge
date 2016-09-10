## How to run the program

If running python 2.7 do the following:
>pip install -r requirements1.txt and python financial_advice.py

If running python 3 do the following:
>pip install -r requirements2.txt and python financial_advice.py


## Financial Advice Application using Machine Learning Algorithms
The following is a detailed implementation of the application. It gives you a stepwise implementation of the code and the API's plugged in. I used the Quandl Financial Dataset API and the Yahoo Financial Dataset API to aid me in getting the fundamental data. I tried adding more API's related to getting the Financial data but it was just a overlap of data. Yahoo Finance and Quandl gave me the right amount of data to build my model.

This application will be to help aid you in the discovery process of companies that might make good long term investments when you buy a companies stock. I analyzed fundamental characteristics of publicly-traded companies (stocks), comparing these fundamentals to the stock's market value performance over time. Our goal is to see if we can use machine learning to identify good stocks with solid fundamentals that matter so we can invest in them. I have used various fundamental stock indicators like Market Cap, Enterprise Value, Trailing P/E, Price/Sales, Price/Book and 33 other features to build my model using various machine learning algorithms which I'll be explaining below. 

With machine learning, everything tends to boil down to features and labels. We have labels, like, in our case, under-performer, and out-performer for each company. With those labels, we have "features" that are the specific values like Debt/Equity ratio that correspond to that label. With that, we're looking to now label our data. To do that, we're going to compare the stock's percentage change to the S&P 500's percentage change. If the stock's percent change is less than the S&P 500, then the stock is and under-performing stock. If the percentage change is more, than the label is out-perform.

## Getting our Data

The quandl api was plugged into the code using the following blob

```python
netIncome = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_NET_INC.csv?&'+endLink).read()
            revenue = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_REV_LAST.csv?&'+endLink).read()
            ROC = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_ROC.csv?&'+endLink).read()

```
The Yahoo Finance dataset was plugged in using the following blob of code:
```python
urlToVisit = 'http://chartapi.finance.yahoo.com/instrument/1.0/'+stock+'/chartdata;type=quote;range=10y/csv'
        stockFile =[]
        try:
            sourceCode = urllib2.urlopen(urlToVisit).read()
            splitSource = sourceCode.split('\n')
            for eachLine in splitSource:
                splitLine = eachLine.split(',')
                if len(splitLine)==6:
                    if 'values' not in eachLine:
                        stockFile.append(eachLine)
        except Exception, e:
            print str(e), 'failed to organize pulled data.'
    except Exception,e:
        print str(e), 'failed to pull pricing data'
```
Once we're able to acquire the data that we're interested in using for our features, we need some way of structuring and organizing our data. I find the Pandas module to be the most useful for this task, given it's quick, efficient and easy data manipulation abilities.

Our end goal with "labeling" data is to categorize it. With investing, we really have just two categories. There's outperform or under-perform. In the future, we may add a third category, "match." For now, we're only interested in whether it would have been better or not to invest in the company, but it might be a good idea to further differentiate companies, like:
>Significantly Outperform Outperform Match (say within 0.5% or something) Under-perform Significantly Under-perform

Then we could convert that to say,
>2 1 0 -1 -2

That said, we're going to keep things simple for now, and just do:
> Outperform (1) Under-perform (0)

This is the S&P 500 data graph of companies:
![alt text](https://s11.postimg.org/jch6w58wj/Screen_Shot_2016_09_09_at_7_32_39_PM.png "Logo Title Text 1")

## How are we doing our classification smartly

With supervised learning, you're going to need to label your data. The idea here is that you can "show" and "teach" the machine some examples of "what is." Then, you can eventually show the machine some data, without the label, expecting it to give you an answer. What we do is feed the data with labels to train the machine. Then we test the machine by feeding information and seeing what the answer is compared to the known answer. If the testing goes well, then we're ready to move forward.
We're looking to now label our data. To do that, we're going to compare the stock's percentage change to the S&P 500's percentage change. If the stock's percent change is less than the S&P 500, then the stock is and under-performing stock. If the percentage change is more, than the label is out-perform.
The most applicable machine learning algorithm for our problem is Linear SVC. The objective of a Linear SVC (Support Vector Classifier) is to fit to the data you provide, returning a "best fit" hyperplane that divides, or categorizes, your data. From there, after getting the hyperplane, you can then feed some features to your classifier to see what the "predicted" class is. This makes this specific algorithm rather suitable for our uses, though you can use this for many situations.

##Linear SVC machine learning and testing our data
![alt text](https://s15.postimg.org/tvlonswij/Screen_Shot_2016_09_09_at_7_46_31_PM.png "Logo Title Text 1")

we're going to cover using two features for machine learning, using Linear SVC with our data. We have many more features to add as time goes on, but we want to use two features at first so that we can easily visualize our data ie Training P/E and Debt/Equity ratio. 

We want to have a function that can build a data-set in a way that is understandable for Scikit-learn.

```python
def Build_Data_Set(features = ["DE Ratio",
                               "Trailing P/E"]):
    netIncome = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_NET_INC.csv?&'+endLink).read()
    revenue = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_REV_LAST.csv?&'+endLink).read()
    ROC = urllib2.urlopen('http://www.quandl.com/api/v1/datasets/OFDP/DMDRN_'+stock.upper()+'_ROC.csv?&'+endLink).read()


    X = np.array(data_df[features].values)

    y = (data_df["Status"]
         .replace("underperform",0)
         .replace("outperform",1)
         .values.tolist())


    return X,y
```

##Screenshots of the Output giving the best companies to invest in

If you notice the code, there is a parameter called how_much_better, which spits out the percentage of companies that are better than the rest that are worth investing in. So if we tweak the value, the output differs from time to time:

1) if how_much_better = 5

![alt text](https://s15.postimg.org/vtgv5hna3/Screen_Shot_2016_09_09_at_8_12_09_PM.png "Logo Title Text 1")

2) if how_much_better = 10

![alt text](https://s10.postimg.org/6v8z13ihl/Screen_Shot_2016_09_09_at_8_13_27_PM.png "Logo Title Text 1")

If you could notice the list keeps changing as the algorithm spits out different companies that are worth investing in. 
