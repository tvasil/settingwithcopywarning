---?color=linear-gradient(180deg, white 50%, #FFD221 50%)
@title[Title Slide]

@snap[west text-bold]
### @color[black](SettingwithCopyWarning)   @fa[tired]
<br>
#### @color[#2D2D2D](Why you sometimes break Pandas and how to deal with it)
@snapend


---
@title[The warning]
# The problem
+++?color=#C2E1C2
@title[Intro to error]

```python3
import pandas as pd
df = pd.read_csv('Xbox 3-day auctions.csv')
df
    auctionid    bid   bidtime          bidder  bidderrate  openbid  price
0  8213034705   95.0  2.927373        jake7870         NaN     95.0  117.5
1  8213034705  115.0  2.943484   davidbresler2           1     95.0  117.5
2  8213034705  100.0  2.951285  gladimacowgirl          58     95.0  117.5
3  8213034705  117.5  2.998947         daysrus          10     95.0  117.5
4  8213060420    2.0  0.065266      donnie4814           5      1.0  120.0

df[df.bidder == 'jake7870']['bidderrate'] = 11

/Users/tania/anaconda3/lib/python3.6/site-packages/
ipykernel_launcher.py:1: SettingWithCopyWarning: A value 
is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead
```

@[1-2](You load your wonderful dataset)
@[3-9](You check it out)
@[5](Oh no! looks like there's a NaN for jake7870)
@[11](You decide to tackle the problem the best way you know)
@[13-16](...until this nasty *Warning* appears!)

---
@title[But this tells you what to do...]
# Wait but... this actually tells you what to do!

+++?color=#C2E1C2
@title[Deep dive]

```python3
df[df.bidder == 'jake7870']['bidderrate'] = 11
/Users/tania/anaconda3/lib/python3.6/site-packages/
ipykernel_launcher.py:1: SettingWithCopyWarning: A value 
is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

df[df.bidder == 'jake7870']['bidderrate']
NaN

df.loc[df.bidder == 'jake7870']['bidderrate'] = 11
df[df.bidder == 'jake7870']['bidderrate']
11.0
```

@[1](Let's try it again!)
@[2-5](the friendly warning)
@[7](Let's check what happened)
@[8](This is definitely not 11)
@[5](Let's try the way that Pandas suggests...)
@[10](...by using .loc)
@[11-12](ok fine, it works)

---

