---
title: Pandas for training
date: 2018-05-22 00:00:00 Z
layout: post
---

In my earlier posts, I shared why pandas is fast. I use pandas for data munging and keras/tensorflow for building DNN models.
After cleaning the data in pandas, when I feed to keras, the training speed was 2x to 4x slower than normal numpy arrays. It's because **pandas uses Fortran order** for its internal numpy arrays. Since **training** happens in batches, it has to **access that data row-wise**. So, the row-access is slower in fortran order and so the training is slower. 

To takle this, I came with some solutions. I was looking into reduce memory usage and also increase speed of training. 
## Method 1 (Convert to C-order numpy array)
After you are done with pandas, create seperate numpy arrays in C-order.
```python
data  = np.ascontiguousarray(df.values[:,:-1])
label = np.ascontiguousarray(df.values[:,:-1])
```
The advantage is that, this conversion happens very fast. It may take ~30 second for 10GB of data.
Cons: It consumes **double the memory** (peak). You can delete the dataframe later to reclaim the space.

## Method 2 (Feather format - exploit)
Whoever is using Python and R often, they would have definitely heard of a file format called feather. It's lightning fast in writing and reading. Since it is new, there is not much features availalble with this format. You can pretty much just write/read with multiple threads and nothing else. 

Anyway, feather is very fast because it saves the data as it is in memory to disk as binary file and reads back. Remember it can store the data in column format only. 
The solution is two-fold.
### 1. Save the transposed data in feather format
`df.transpose()` won't take much time. But writing the data to feather format of transposed dataframe will be slower (~4x). The file size is same. 

```python
import feather
feather.write_dataframe(df.transpose().reset_index(), 'data_label.feather')
```


### 2. Read feather and transpose
Now read the file and  transpose

```python
import feather
df=feather.read_dataframe('data_label.feather', nthreads=4)
df.set_index("index", drop=True, inplace=True)
df=df.transpose()
data = df.values[: , :-1]
label = df.values[: ,  -1]
np.info(data)
np.info(label)
```

Since all the operations here done on reference, it does not incur any additional memory. Now `df.values` is in C contiguous order. Training using `data` and `label` numpy arrays will not be slower. 

Because of feather format limitations, I have to reset_index and set_index. feather format does not store index names. It will be fixed in [future](https://github.com/wesm/feather/issues/200).
Once the memory map file feature support comes for feather, this will enable to run training on very large data.