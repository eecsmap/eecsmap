# ucb data 100: Principles and Techniques of Data Science

> course site: https://ds100.org/
> 
> resources: https://github.com/DS-100


* jupyter shortcuts:
  - `m`: convert cell to `markdown`
  - `y`: convert cell to `code`
  - `%%time` to measure the time for running a cell
* read data from csv file：`pandas.read_csv()`
* use the 538 style in plot (including grids, fonts, etc): `plt.style.use('fivethirtyeight')`
* `range` and `np.arange` default to be `[start, end)`, with default step `1`
* common imports
  ```python
  import numpy as np
  import pandas as pd
  import matplotlib.pyplot as plt
  ```
* **histogram** key concepts:
  - domain of possible sample values is splited into bins, intervals as `[begin, end)` except the last one which is `[begin, end]`
  - to indicate `n` bins, we use `n + 1` split points
  - to normalize the total area as `1`, use `density=True`
  - to draw the split lines between bins, use `ec=white` or color of your choice
  - example:
    ```python
    plt.hist([random.randint(1,6) for i in range(50000)], bins=bin_values, ec='white', density=True)
    ```
  - generate hitogram: `np.histogram(data)`
  - plot histogram: `plt.hist(data)`
* pandas data operations:
  - `data.shape`
  - `data.head(n)`
  - group by values of given column: `data.groupby(column_name).count()`
  - filter data using a column predicate: `data[data[column_name]][target_column]`
  - Every column selected is a `Series`
  - To select multiple columns: `data[[column_1, column_2]]`
  - `count()` excludes `NA`, whereas `size()` includes `NA`
  - `df.sort_values(by=[], ascending=False)`
* term:
  - random permutation & permutation test
  - A/B test
  - chance variation
 
* ref:
  - pandas: https://pandas.pydata.org/docs/getting_started/index.html
  - 
