# Deep-Learning-Basics

## 🚚 Loading Data: **tf.data API**

### Concept of tf.data
The tf.data API enables you to build complex input pipelines from simple, reusable pieces. tf.data also makes it possible to handle large amount of data, reading from different data formats, and perform complex transformation

### What is the Tensorflow Input Pipeline?
The input pipeline is a quick and easy utility provided in tf.dataapi to make complex input pipelines from simple and reusable codes and all in few lines of code. It also allows handling a large amount of data, thus giving low-end machines an advantage in computing them.
It does it by wrapping the data into tf.data.dataset class and performing a series of operations on them called ETL - Extract, Transform, Load.

#### The benefits of using **Tensorflow Input Pipeline** are as follows:
1. ***`Loading data`***: Data is loaded in chunks in tf.data.dataset structure or batches, **allowing huge data to be managed and loaded while retaining memory space**. Also, It allows support for different data formats and types including cloud storage (S3).
2. ***`Data manipulation/ augmentation`***:- In general this is done with Pandas, Numpy -text / tabular/ numerical data, or Keras ImageDataGenerator / OpenCV-image data. But here, it can be performed in the input pipeline itself, **allowing faster and rapid prototyping while coding**. **Filtering**, **mapping**, **resizing**, **cropping** are to name a few.
3. ***`One Liner Finally`***: All this can be done in a single line of code thus saving memory space. Here is a quick snippet of it along with an explanation:
4. ***`Distributed Computing`***: It **supports distributed and parallel computing** for enterprises which is essential for cloud computing and big data.

`Note:-` Tensors are the underlying data structures behind tf.data.dataset, so it's different from NumPy arrays or Pandas dataframe. Don’t get mistaken😉.

### How to optimise pipeline?
There are several ways how tf.data reduce computational overhead which can be easily implemented into your pipeline:

1. Prefetching
2. Parallelising data extraction
3. Parallelising data transformation
4. Caching
5. Vectorised mapping

#### 0. Naive approach
Before we start on these concepts, we will have to first understand how the naive approach works when a model is being trained.

![1_NlsHwaUrEsGII-o4LAVWsQ.png](attachment:7eee140a-0eec-4cf1-9fb1-6f59739dc192.png)
This diagram shows that a training step includes opening a file, fetching a data entry from the file and then using the data for training. We can see clear inefficiencies here as when our model is training, the input pipeline is idle and when the input pipeline is fetching the data, our model is idle.

tf.data solves this issue by using prefetching .

#### 1. Prefetching
Prefetching solves the inefficiencies from naive approach as it aims to overlap the preprocessing and model execution of the training step. In other words, when the model is executing training step n, the input pipeline will be reading the data for step n+1.

The tf.data API provides the tf.data.Dataset.prefetch transformation. It can be used to decouple the time when data is produced from the time when data is consumed. In particular, the transformation uses a background thread and an internal buffer to prefetch elements from the input dataset ahead of the time they are requested.


![1_PISwE0ow6bjhlNME_Uq6Yw.png](attachment:24758ece-0054-4c9d-8c3a-5557b8bc555c.png)

There is an argument that prefetch transformation requires — the number of elements to prefetch. However, we could simply make use of tf.data.AUTOTUNE— provided by tensorflow, which prompts tf.data runtime to tune the value dynamically at runtime.


#### 2. Parallelising data extraction
There exists computational overhead when raw bytes are loaded into memory when reading data, as it may be necessary to deserialise and decrypt the read data. This overhead exists irrespective of whether data is stored locally or remotely.

![1_Amms5OWLomjOe0MJnnQpJw.png](attachment:f90934f0-a897-4f33-9114-1d46c24d3546.png)

![1_JGSJ-Ax35uNHAQ9kQeSdQw.png](attachment:f7838ed3-3a08-46d2-99d2-12c523f3fb18.png)

To deal with this overhead, tf.data provides tf.data.Dataset.interleave transformation to parallelise the data loading step, interleaving the contents of other datasets.

Similarly, this interleave transformation supports tf.data.AUTOTUNE which will again delegate the decision of the level of parallelism during tf.data runtime.

#### 3. Parallelising data transformation
In most scenarios, you will have to do some preprocessing to your dataset before passing it to the model for training. The tf.dataAPI takes care of this by offering the tf.data.Dataset.map transformation — which applies a user-defined function to each element of the input dataset.

Because input elements are independent of one another, the pre-processing can be parallelised across multiple CPU cores.

![1_Sbk_IxtsPuUTCV9yQq1ZBA.png](attachment:474cc40e-5173-41ec-bf39-eee6ff7c0c69.png)
![1_eKf7t8tUgZFIsmxNvuQA0w.png](attachment:f09ccb99-3bea-4e62-a38e-43178e7cd218.png)

To utilise multiple CPU cores, you will have to pass in num_parallel_calls argument to specify the level of parallelism you want. Similarly, the map transformation also supports tf.data.AUTOTUNE which will again delegate the decision of the level of parallelism during tf.data runtime.

#### 4. Caching
![1_Jt3WrFTO22qmKoj-eKO5xA.png](attachment:fe6105bf-d738-4fe3-814d-c9becc114b58.png)

tf.data also have caching abilities with tf.data.Dataset.cache transformation. You can either cache a dataset in memory or in local storage. The rule of thumb will be to cache a small dataset in memory and a large dataset in local storage. This thus saves operation like file opening and data reading from being executed during each epoch — next epochs will reuse the data cached by the cache transformation.

One thing to note — you should cache after preprocessing (especially when these preprocessing functions are computational expensive) and before augmentation, as you would not want to store any randomness from your augmentations.

#### 5. Vectorised mapping
![1_wOa4Zv3S_bpZL4eLtBqH1g.png](attachment:2e037197-9623-400a-a3db-9fe58c8bfb32.png)

When using the tf.data.Dataset.map transformation as mentioned previously under ‘parallelising data transformation’, there is certain overhead related to scheduling and executing the user-defined function. Vectorising this user-defined function — have it operate over a batch of inputs at once — and applying the batch transformation before the map transformation helps to improve on this overhead.

#### Conclusion
In summary, you can use prefetch transformation to overlap the work done by pipeline (producer) and the model (consumer), interleave transformation to parallelise data reading, map transformation to parallelise data transformation, cache transformation to cache data in memory or local storage and also vectorising your map transformations with the batch transformation.
As mentioned at the start, one of the worst things to experience is seeing your GPU capacity not fully utilised with the bottleneck on the CPU. With tf.data , you will most probably be happy with your GPU utilisation!
