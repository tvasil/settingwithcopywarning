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
+++?color=#e8f9e8
@title[Intro to the warning]
@snap[north-west span-90] 
#### Let's consider a simple example
@snapend

```python
>>> import pandas as pd
>>> df = pd.read_csv('xbox_ebay_auctions.csv')
>>> df.head()
    auctionid    bid   bidtime          bidder  bidderrate  openbid  price
0  8213034705   95.0  2.927373            jake         NaN     95.0  117.5
1  8213034705  115.0  2.943484           david           1     95.0  117.5
2  8213034705  100.0  2.951285  gladimacowgirl          58     95.0  117.5
3  8213034705  117.5  2.998947             rus          10     95.0  117.5
4  8213060420    2.0  0.065266          donnie           5      1.0  120.0

>>> df[df.bidder == 'jake']['bidderrate'] = 11

SettingWithCopyWarning: A value is trying to be set 
on a copy of a slice from a DataFrame. 
Try using .loc[row_indexer,col_indexer] = value instead
```
@[1-2](You load your dataset)
@[3-9](You check it out)
@[5](Oh no! looks like there's a NaN for jake)
@[11](You decide to tackle the problem the best way you know)
@[13-15](Until this warning appears)
@[15](But wait! It tells us what to do!)

+++?color=#e8f9e8
@title[Let's fix it]
@snap[north-west span-90] 
#### Let's fix it
@snapend

```python
>>> df.loc[df.bidder == 'jake','bidderrate'] = 11
# Notice the surprising lack of warnings! :D

>>> df[df.bidder == 'jake']['bidderrate']
11.0
```

@[1](Let's try it again with *.loc*!)
@[2](Now this gives us no warning)
@[4-5](Ok fine, it works as expected)

+++?color=white
@title[Temporary Victory]
### Victory!
![Victory](https://media.giphy.com/media/JlVb5LVZ4y5Uc/giphy.gif)

---
@title[The actual problem]
@css[text-h3](The actual problem)
@css[text-h3 fragment](is all about *view* vs *copy*)
@css[text-h3 fragment](and *chained assignment*)

+++?color=#e8f9e8
@title[View vs copy]

@snap[north-west]
#### View vs Copy
@snapend

@img[fragment](https://www.dataquest.io/wp-content/uploads/2019/01/modifying.png)


+++?color=#e8f9e8
@snap[north-west]
#### What happens when we use *[]*?
@snapend

```python
>>> df[df.bidder == 'jake']['bidderrate'] = 11

SettingWithCopyWarning: A value is trying to be set 
on a copy of a slice from a DataFrame. Try using 
.loc[row_indexer,col_indexer] = value instead

>>> df[df.bidder == 'jake']['bidderrate']
NaN
```
@[1-5](While we thought we're modifying the original dataframe...)
@[7-8](Only an intermediary object -- a copy! -- is actually modified)

+++?color=#e8f9e8
@title[What happens under the hood]
@snap[north-west]
#### Chained assignment
@snapend

```python
# Code you write
df[df.bidder == 'jake']['bidderrate'] = 11

# Code that is executed
df.__getitem__(df.__getitem__('bidder') == 'jake')
.__setitem__('bidderrate', 11)

# IS IT A COPY? A VIEW?
```

@[1-2](Square bracket notation)
@[4-6](Actually executes both a **get** and a **set** operation)
@[5](What is this object?)
@[1-8](nobody knows)

+++?color=#e8f9e8
@title[But if you use .loc]

@snap[north-west]
#### and now with *.loc* notation
@snapend

```python
# Code you write
df.loc[df.bidder == 'jake','bidderrate'] = 11

# Code that is executed
df.loc.__setitem__((df.__getitem__('bidder') == 'jake',
'bidderrate'), 11)

# THIS IS GUARANTEED TO CHANGE THE ORIGINAL DF
```
@[1-2](*.loc* notation)
@[4-6](Only executes a **set** operation)
@[1-8](the only slicing *guaranteed* to return a view and to refer to the original dataframe))

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
@[1](If you're only going to work on `winners` from now on, make an explicit copy)

+++
@title[Solution3]

@snap[north-west] 
### Solutions
#### 3. If you want to change the original dataframe, do that directly
@snapend

```python
## instead of this
winners = df.loc[df.bid == df.price]
winners.loc[df.bidder == 'jake', 'bidderrate'] = 11

## do this!
df.loc[df.bidder == 'jake', 'bidderrate'] = 11
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




