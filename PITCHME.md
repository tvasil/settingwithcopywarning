---?color=linear-gradient(180deg, white 50%, #FFD221 50%)
@title[Title Slide]

@snap[west text-bold]
### @color[#2D2D2D](`SettingwithCopyWarning`)   @fa[tired]
<br>
#### @color[#2D2D2D](Why you sometimes break Pandas and how to deal with it)
@snapend


---
@title[The warning]
# The problem
+++?color=#C2E1C2
@title[Intro to error]
@snap[north span-90] 
#### Let's consider a simple example
@snapend

```python
>>> import pandas as pd
>>> df = pd.read_csv('Xbox 3-day auctions.csv')
>>> df.head()
    auctionid    bid   bidtime          bidder  bidderrate  openbid  price
0  8213034705   95.0  2.927373        jake7870         NaN     95.0  117.5
1  8213034705  115.0  2.943484   davidbresler2           1     95.0  117.5
2  8213034705  100.0  2.951285  gladimacowgirl          58     95.0  117.5
3  8213034705  117.5  2.998947         daysrus          10     95.0  117.5
4  8213060420    2.0  0.065266      donnie4814           5      1.0  120.0

>>> df[df.bidder == 'jake7870']['bidderrate'] = 11

/Users/tania/anaconda3/lib/python3.6/site-packages/
ipykernel_launcher.py:1: SettingWithCopyWarning: A value 
is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead
```

@[1-2](You load your dataset)
@[3-9](You check it out)
@[5](Oh no! looks like there's a NaN for jake7870)
@[11](You decide to tackle the problem the best way you know)
@[13-16](Until this warning appears)
@[16](But wait! It tells us what to do!)

+++?color=#C2E1C2
@title[Let's fix it]
@snap[north span-90] 
#### Let's fix it
@snapend

```python
>>> df.loc[df.bidder == 'jake7870','bidderrate'] = 11
# Notice the surprising lack of warnings! :D

>>> df[df.bidder == 'jake7870']['bidderrate']
11.0
```

@[1](Let's try it again with *.loc*!)
@[2](Now this gives us no warning)
@[4-5](Ok fine, it works as expected)

---
@title[Temporary Victory]
### Victory!
![Victory](https://media.giphy.com/media/JlVb5LVZ4y5Uc/giphy.gif)

---
@title[Not so fast]
### not so fast...
![Cersei](https://media.giphy.com/media/VIu9jzc36itzy/giphy.gif)


+++?color=#C2E1C2
@title[Chained assignment]
@snap[north span-95] 
#### A more realistic example
@snapend

```python
>>> winners = df.loc[df.bid == df.price]
##a lot of other important stuff 
##(200 lines of code)
>>> winners.tail()
      auctionid     bid   bidtime         bidder  bidderrate  openbid   price
514  8215571039  150.00  2.997442     lambonius1           0    50.00  150.00
515  8215571039  150.00  2.998437   analogkidd99           0    50.00  150.00
531  8215582227  152.50  2.999664  ultimatum_man           2    60.00  152.50
542  8215605488   61.00  2.586493     pgauctions          16     0.99   61.00
555  8215610555   35.09  2.973264       biquÃni4         114     5.00   35.09

>>> winners.loc[555, 'bidder'] = 'biquini4'
```

@[1](Let's say now you want to only look at the auction winners)
@[2-3](You create a new dataframe and forget about the old one, while working on other code)
@[4-10](Suddenly you check out the data again and see there was some encoding error)
@[10-12](This really bothers you right now, so you just go ahead and replace it with *.loc*)

+++?color=#C2E1C2
@title[Chained assignment2]
@snap[north span-90] 
#### A more realistic example
@snapend

```python3
/Users/tania/anaconda3/lib/python3.6/site-packages/pandas/
core/indexing.py:543: SettingWithCopyWarning: A value is 
trying to be set on a copy of a slice from a DataFrame.Try 
using .loc[row_indexer,col_indexer] = value instead

```
@[1-4](but you get the warning again!)


+++?color=#C2E1C2
@title[But I used .loc]

### But I used .loc...
![why](https://media.giphy.com/media/1M9fmo1WAFVK0/giphy.gif)

---
@title[The problem]
@css[text-h3](The actual problem)
@css[text-h3 fragment](is all about *view* vs *copy*)
@css[text-h3 fragment](and *chained assignment*)

+++?color=#C2E1C2
@title[View vs copy]
#### View vs Copy
@img[fragment](https://www.dataquest.io/wp-content/uploads/2019/01/modifying.png)

+++?color=#C2E1C2
@title(What happens in the original example)
@snap[north span-90] 
#### So what happened with *[]*?
@snapend

```python
>>> df[df.bidder == 'jake7870']['bidderrate'] = 11

/Users/tania/anaconda3/lib/python3.6/site-packages/
ipykernel_launcher.py:1: SettingWithCopyWarning: A value 
is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

>>> df[df.bidder == 'jake7870']['bidderrate']
NaN
```
@[1-6](While you think you're modifying the original df...
@[8-9](Only an intermediary object is actualy being modified)

+++?color=#C2E1C2
@title[What happens under the hood]
#### Chained assignment

```python
# Code you write
df[df.bidder == 'jake7870']['bidderrate'] = 11

# Code that is executed
df.__getitem__(df.__getitem__('bidder') == 'jake7870')
.__setitem__('bidderrate', 11)

# IS THIS A COPY? A VIEW?
```

@[1-2](Square bracket notation)
@[4-6](Actually executes both a **get** and a **set** operation)
@[5](What is `__getitem__(data.__getitem__('bidder') == 'jake7870'`)?)
@[8](nobody knows)

+++?color=#C2E1C2
@title[But if you use .loc]
#### and now with *.loc* notation

```python
# Code you write
df.loc[df.bidder == 'jake7870','bidderrate'] = 11

# Code that is executed
df.loc.__setitem__((df.__getitem__('bidder') == 'jake7870',
'bidderrate'), 11)

# THIS IS GUARANTEED TO BE A VIEW
```
@[1-2](*.loc* notation)
@[4-6](Only executes a **set** operation)
@[8]((the only one *guaranteed* to be a view and refer to the original dataframe))

+++?color=#C2E1C2
@title[Locations in memory]
#### If you don't trust me, check the locations in memory

```python
>>> id(df[df.bidder == 'jake7870']['bidderrate'])
4638136584

>>> id(df.loc[df.bidder == 'jake7870', 'bidderrate'])
112051366544
```
@[1-2](Location of intermediate object)
@[4-6](Location of the view)

---
@title[Solution]
@snap[north] 
@css[text-h2 text-bold](Solutions <br>)
@snapend
@css[text-h3 fragment](1. Use *.loc* to replace values <br>)

@fa[thumbs-up fa-4x fragment]

---
@title[Solution2]
@snap[north] 
@css[text-h2 text-bold](Solutions <br>)
@snapend
@css[text-h3](2. Make sure to make a copy when you really mean to! <br>)

```python
winners = df.loc[df.bid == df.price].copy()
```
@[1](If you're only gonna work on `winners` from now on, make an explicit copy!)

---
@title[Solution3]
@snap[north] 
@css[text-h2 text-bold](Solutions <br>)
@css[text-h3](3. If you want to change the origal dataframe, do that directly)

```python
data.loc[data.bidder == 'jake7870','bidderrate'] = 11
```
@[1](Again, change by using *.loc*)

---
@title[Suppress warning]
@snap[north] 
@css[text-h2 text-bold](Should I suppress the warning? <br>)
@css[text-h3](My suggestion is NO)

```python
pd.set_option('mode.chained_assignment', None)

pd.set_option('mode.chained_assignment', 'raise')
```
@[1](If you're really know what you're doing, you can suppress the warning)
@[3](Or elevate it to an exception if you're writing production code and want to catch this issue)

---?color=#9FEAA7
@title[Thank you]
## Thank you!

@snap[west span-10]
![github](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png) /tvasil
![twitter](https://image.flaticon.com/icons/png/512/23/23681.png) twitter.com/tvasi
![pyladies](https://cdn-images-1.medium.com/max/1600/1*Pc8QzOC5RNvDaXNGzzRWWg.png) @tania
@snapend



