---
title: File formats for pandas Dataframe
layout: post
---

# File formats
I had to store many GB of data for training.  Data source is csv. To avoid parsing csv everytime during my training, I explored different file formats, namely
1.  feather
2.  hdf
3.  numpy
4.  parquet
5.  csv

Each file format has its pros and cons. 

## Code
I used 320MB data to test the fileformat performances. Here is the script I used to evaluate
```python
In [1]: import pandas as pd

In [2]: PATH="/home/karthikeyan/data."

In [4]: %timeit df=pd.read_feather(PATH+"feather")
275 ms ± 2.56 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [5]: %timeit df=pd.read_hdf(PATH+"hdf")
254 ms ± 11.7 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [7]: %timeit df=pd.read_parquet(PATH+"pqt")
1.61 s ± 88.8 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [9]: %timeit df=pd.read_csv(PATH+"csv")
20.9 s ± 1.3 s per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [10]: df=pd.read_parquet(PATH+"pqt")

In [11]: df.info(memory_usage="deep")
#<class 'pandas.core.frame.DataFrame'>
Int64Index: 41208 entries, 0 to 41207
Columns: 4006 entries, LevelFI0 to Hinst
dtypes: object(1), uint16(4005)
memory usage: 319.9 MB


In [12]: np.save(PATH+"npy", df.values)

In [13]: %timeit df = pd.DataFrame(np.load(PATH+"npy"))
8 s ± 106 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [14]: np.save(PATH+"npy2", df.transpose().values)

In [15]: %timeit df = pd.DataFrame(np.load(PATH+"npy2.npy").transpose())
5.55 s ± 123 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

In [16]: df.shape
Out[16]: (41208, 4006)

```

## Comparison
Ofcourse there are plenty of blogs comparing the file format performance. But I needed to analyse it myself for the size of data that I deal with (~10's GB).
It's important to note theat my 320 MB data contains unsigned integer and also a column of Strings.

FORMAT          | File Size | Time Taken to read || Pros                                                                                | Cons                                                          
-----|-----:|-----:||-----|-----
feather         |     318MB |             275 ms || Very Fast Save                                                                          | File size similar as csv                                      
                         |                        |                              || Read Good for columnar data                                         | 
hdf             |     319MB |             254 ms || Fairly Fast Save                                                    | File size similar as csv                                       
                     |                        |                              || Very Fast Read                                                    |   
numpy           |     319MB |            8000 ms || Flat format                                                                         | File size similar as csv                                      
numpy transpose |     319MB |            5500 ms || Flat format                                            |
                     |                        |                              || Medium Read                                                            | File size similar as csv                                      
parquet         |      77MB |            1610 ms || Fairly Fast Save (4x feather)                       |
|                        |                              || Fairly Fast Read (6x feather) File size is much small | Not widely used                                               
csv             |     337MB |           20900 ms || Human readable                                                                      | Large filesize Slow reading/parsing 
                     |                        |                              ||                                                                                                      | No good for quick queries 

<br>
Overall winner is **`parquet`** in terms of tradeoff between speed and filesize.
But if you want to incrementally add/query data, `hdf` is recommended. 
If you are looking at plain speed, use *`feather`*. It's very fast and optimum for column data processing. I have a trick to [trick ]({{ site.baseurl }}/pandas-for-training/)

### NOTE:
Transpose trick is something I use often to speedup reading and writing data to dataframe quickly.