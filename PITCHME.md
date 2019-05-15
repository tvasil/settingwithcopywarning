---?color=linear-gradient(180deg, white 50%, #FFD221 50%)
@title[Intro]

@snap[north-west span-30]
![pandas](assets/img/pandas_logo.png)
@snapend

@snap[west text-bold]
### @color[#2D2D2D](`SettingwithCopyWarning`)   @fa[tired]
<br>
#### @color[#2D2D2D](Why you sometimes break Pandas and how to deal with it)
@snapend


---
@title[The warning]
# The problem
+++?color=#E0E2DB
@title[Intro to the warning]
@snap[north-west span-90] 
#### Let's consider a simple example
@snapend

```
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
@snap[north-west span-90] 
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

+++?color=white
@title[Temporary Victory]
### Victory!
![Victory](https://media.giphy.com/media/JlVb5LVZ4y5Uc/giphy.gif)

+++?color=white
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

555  8215610555   35.09  2.973264       biquÃni4         114     5.00   35.09

>>> winners.loc[555, 'bidder'] = 'biquini4'
```

@[1](Let's say now you want to only look at the auction winners)
@[3-4](You create a new dataframe and forget about the old one, while working on other code)
@[6](Then you suddenly come across encoding error, and it really bothers you)
@[8](so you just go ahead and replace it with *.loc*)

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
@title[The actual problem]
@css[text-h3](The actual problem)
@css[text-h3 fragment](is all about *view* vs *copy*)
@css[text-h3 fragment](and *chained assignment*)

+++?color=#C2E1C2
@title[View vs copy]

@snap[north-west]
#### View vs Copy
@snapend

@img[fragment](https://www.dataquest.io/wp-content/uploads/2019/01/modifying.png)


+++?color=#C2E1C2
@snap[north-west]
#### What happens when we use *[]*?
@snapend

```
>>> df[df.bidder == 'jake7870']['bidderrate'] = 11

/Users/tania/anaconda3/lib/python3.6/site-packages/
ipykernel_launcher.py:1: SettingWithCopyWarning: A value 
is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

>>> df[df.bidder == 'jake7870']['bidderrate']
NaN
```
@[1-6](While we thought we're modifying the original dataframe...)
@[8-9](Only an intermediary object is actually modified)

+++?color=#C2E1C2
@title[What happens under the hood]
@snap[north-west]
#### Chained assignment
@snapend

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
@[5](What is this object?)
@[8](nobody knows)

+++?color=#C2E1C2
@title[But if you use .loc]

@snap[north-west]
#### and now with *.loc* notation
@snapend

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
@[8](the only slicing *guaranteed* to return a view and to refer to the original dataframe))

---
@title[Solution]

@snap[north-west] 
### Solutions
#### 1. Use *.loc* to replace values <br>
@snapend

@snap[midpoint span-100]
@fa[thumbs-up fa-4x fragment]
@snapend
+++
@title[Solution2]

@snap[north-west] 
### Solutions
#### 2. Make sure to create a copy when you really mean to!
@snapend

```python
winners = df.loc[df.bid == df.price].copy()
```
@[1](If you're only going to work on `df2` from now on, make an explicit copy!)

+++
@title[Solution3]

@snap[north-west] 
### Solutions
#### 3. If you want to change the original dataframe, do that directly
@snapend

```python
## instead of this
winners = df.loc[df.bid == df.price]
winners.loc[df.bidder == 'jake7870', 'bidderrate'] = 11

## do this!
df.loc[df.bidder == 'jake7870', 'bidderrate'] = 11
```
@[1-3](DON'T do this!)
@[5-6](Do *this* instead!)

+++
@title[Should I suppress warnings?]

@snap[north-west] 
### Extra: should I suppress the warning?
@snapend

@css[text-h4 fragment](For the love of Pandas)
@css[text-h4 fragment](**NO**)


```python
pd.set_option('mode.chained_assignment', None)

pd.set_option('mode.chained_assignment', 'raise')
```
@[1](If you really know what you're doing, you can suppress the warning)
@[3](or elevate it to an exception if you're writing production code and want to catch this issue)

---?color=#FFD221
@title[Stay in touch!]

@snap[west contact-links]
@css[thank-you](Thank you!)<br>
<a href="https://twitter.com/tvasi">
@fa[twitter-square pad-right-icon]@css[twitter-handle](@tvasi)
</a><br>
<a href="https://github.com/tvasil/settingwithcopywarning">
@fa[github-square pad-right-icon]@css[git-handle](tvasil/settingwithcopywarning)
</a><br>
@snapend

@snap[east span-50]
![stayintouch](assets/img/contact-1.png)
@snapend



