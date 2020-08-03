+++ 
title = "Using Microsoft Excel Power Query to Clean and Sort Data during COVID-19" 
author = "Jason Geslois" 
date = "2020-08-02" 
+++

Not too long ago, I discovered a very useful tool that Microsoft created for Excel called Power Query. You can find and use it from under the Data tab within the ribbon at the top, and it can do quite a lot in terms of data wrangling and transformation for a variety of tasks. It has helped me tremendously in working and getting out variations of different data for different groups of people during the COVID-19 pandemic. 
In this post, I wanted to share a few helpful tools that have not only been useful to me, but also might be helpful to other health departments working with data in Excel for daily case counts. I am working this example on a fake disease dataset that I created, where each row can be conceptualized as one individual entry of disease count by day (ex. John Smith, Positive for COVID) and each column is a different feature that describes it (ex. GENDER, AGE, RESIDENCE) etc. 

![Example Data](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query01.jpg) 
{{< figure library="1" src="query01.jpg" title="" lightbox="true" >}}

So, for this post, I want to complete three tasks. 
-	We will clean a column of data by converting it to UPPER CASE as well as trimming any white space before or after it.
  *	Why is this useful? for various analyses as well as finding duplicates in different entries where there may be errors. Ex. You have 3 John Smith entries by each name with each looking like John, JOHN, john_. These may be hard to find if your dataset is very large. 
-	We will add a new conditional column based on values in another column.
  *	Why is this useful? Can create descriptive categorical data for grouping or associating different variables to understand your data better.
-	And finally, we will filter to only view new entries of data showing the current day’s date.
  *	Why is this useful? Suppose you are entering a lot of new data each day from multiple sources or multiple people, and you need to share certain elements of this, and only want to view the daily new entries, this will populate each time you refresh for the current day. 
So to start, lets load in our fake disease data file, but we do not want to open it the usual way, we want to use Power Query to bring it in and then be able to work with it to do these things. So, let us go to the data ribbon, and click “Get Data” and as our file is in another Excel workbook, we will click that. As you can see, Excel can import a large variety of data types from other data sources. 

![Data Types](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query02.jpg) 
{{< figure library="1" src="query02.jpg" title="" lightbox="true" >}}

##Cleaning the Column of Data

Once the file is loaded, the Power Query Editor window opens. From here, click the Transform tab at the top and you can see the Format button where both our Upper-case tool and Trim tools are along with a few other handy ones. Select the top of the column header that we want to begin to clean and click each one of these tools. 

![Tools](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query03.jpg) 
{{< figure library="1" src="query03.jpg" title="" lightbox="true" >}}

If you made a mistake or need to undo, over on the right-hand side is the query settings showing both steps completed. You can just click on the one that you do not need, and then click the X on the left side and it will undo that step. 

![Undo](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query04.jpg) 
{{< figure library="1" src="query04.jpg" title="" lightbox="true" >}}

##Using Conditional Columns

In this example what we want to do is, create a conditional column to label your data based on specific conditions you may have. For example lets say you have to do certain quality assurance checks on each of the case reports of diseases each year and you want to label it to be able to know how many more you have left to process after you have done your qualitative reviews. So, in this example, you have finished with Pertussis and you want to label each one as being “Processed” and all the other conditions as “Incomplete.”  

![Labels](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query05.jpg) 
{{< figure library="1" src="query05.jpg" title="" lightbox="true" >}}

When you click the conditional column button, it brings up another window where you can set multiple conditions on what you want the new column to be based on. Below, I have already filled out how we want to sort and create this. So we are naming our new column “Status” and basically saying the following: If within the “Illness” column, the value is “Pertussis”, then show in the new column the word “Processed”, if not, then show “Incomplete.” 

![Conditional](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query06.jpg) 
{{< figure library="1" src="query06.jpg" title="" lightbox="true" >}}

You can make this as simple or as complex as you need by adding clauses and many other additional conditions, changing the Operator to show if something “does not equal”, or “if it contains” a value etc. Below is the new column and its output. 

![Clauses](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query07.jpg) 
{{< figure library="1" src="query07.jpg" title="" lightbox="true" >}}

##Filtering for Today’s Date

Our last example will be to look at only keeping today’s dated entries. So, I am going to look at the “DateReported” column I have with the date values, and I am going to click the arrow to the right, then Date Filters, Day, and then “Today.” This again will only load the current day’s values. You can see there is a variety of other date and range options you can play with. But in this example, if tomorrow, you were to run the query again, it would repopulate your entries for tomorrow’s date as it would be the new “Today.”  

![Dates](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query08.jpg) 
{{< figure library="1" src="query08.jpg" title="" lightbox="true" >}}

And to finish out, select Close & Load on the home tab, and you will see a new sheet open with the data you have selected only showing Today’s entries. 

![Save](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query09.jpg) 
{{< figure library="1" src="query09.jpg" title="" lightbox="true" >}}

Each new day you do not need to do anything in Power Query but can open your file, and then go to Data tab, and click “Refresh All” and it will load. If you find that it loaded blank, then there would not be any entries for that day. 

![Refresh](https://github.com/jasongeslois/jasongeslois.com-site/blob/master/content/post/query10.jpg) 
{{< figure library="1" src="query10.jpg" title="" lightbox="true" >}}


When I first learned about Power Query, I was excited to learn about these and the many other features it has to offer. It adds a new set of convenience and abilities to just using Excel for data tasks as it is ubiquitous across most business and public health environments especially when there may not be funds to buy specific software or the trained data staff that know coding for other types of software to be able to do certain data cleaning tasks. 

