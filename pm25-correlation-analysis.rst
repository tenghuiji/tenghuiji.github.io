Title: PM2.5 Correlation Analysis
Date: 2010-12-04 10:20
Category: Correlation Analysis
# 北京PM2.5相关性分析


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```

    /Users/hjteng/anaconda3/envs/TensorFlow/lib/python3.6/site-packages/statsmodels/compat/pandas.py:56: FutureWarning: The pandas.core.datetools module is deprecated and will be removed in a future version. Please use the pandas.tseries module instead.
      from pandas.core import datetools


#### 加载原始数据


```python
dataset = pd.read_csv('./dataset/beijing.csv')
dataset.head(5)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>No</th>
      <th>year</th>
      <th>month</th>
      <th>day</th>
      <th>hour</th>
      <th>pm2.5</th>
      <th>DEWP</th>
      <th>TEMP</th>
      <th>PRES</th>
      <th>cbwd</th>
      <th>Iws</th>
      <th>Is</th>
      <th>Ir</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2010</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>-21</td>
      <td>-11.0</td>
      <td>1021.0</td>
      <td>NW</td>
      <td>1.79</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2010</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>NaN</td>
      <td>-21</td>
      <td>-12.0</td>
      <td>1020.0</td>
      <td>NW</td>
      <td>4.92</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2010</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>NaN</td>
      <td>-21</td>
      <td>-11.0</td>
      <td>1019.0</td>
      <td>NW</td>
      <td>6.71</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2010</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>NaN</td>
      <td>-21</td>
      <td>-14.0</td>
      <td>1019.0</td>
      <td>NW</td>
      <td>9.84</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>2010</td>
      <td>1</td>
      <td>1</td>
      <td>4</td>
      <td>NaN</td>
      <td>-20</td>
      <td>-12.0</td>
      <td>1018.0</td>
      <td>NW</td>
      <td>12.97</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



No: row number   
year: year of data in this row   
month: month of data in this row   
day: day of data in this row   
hour: hour of data in this row   
pm2.5: PM2.5 concentration (ug/m^3)   
DEWP: Dew Point (â„ƒ)   
TEMP: Temperature (â„ƒ)   
PRES: Pressure (hPa)   
cbwd: Combined wind direction   
Iws: Cumulated wind speed (m/s)   
Is: Cumulated hours of snow   
Ir: Cumulated hours of rain

#### 数据清洗


```python
dataset = pd.read_csv('./dataset/beijing.csv',header=0,parse_dates=[[1,2,3,4]],index_col=0,
                      date_parser=lambda date: pd.datetime.strptime(date,'%Y %m %d %H'))
dataset.drop('No',axis=1,inplace=True)
dataset.index.name='date'
dataset.columns = ['pollution', 'dew', 'temp', 'press', 'wnd_dir', 'wnd_spd', 'snow', 'rain']
dataset.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pollution</th>
      <th>dew</th>
      <th>temp</th>
      <th>press</th>
      <th>wnd_dir</th>
      <th>wnd_spd</th>
      <th>snow</th>
      <th>rain</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-01 00:00:00</th>
      <td>NaN</td>
      <td>-21</td>
      <td>-11.0</td>
      <td>1021.0</td>
      <td>NW</td>
      <td>1.79</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2010-01-01 01:00:00</th>
      <td>NaN</td>
      <td>-21</td>
      <td>-12.0</td>
      <td>1020.0</td>
      <td>NW</td>
      <td>4.92</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2010-01-01 02:00:00</th>
      <td>NaN</td>
      <td>-21</td>
      <td>-11.0</td>
      <td>1019.0</td>
      <td>NW</td>
      <td>6.71</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2010-01-01 03:00:00</th>
      <td>NaN</td>
      <td>-21</td>
      <td>-14.0</td>
      <td>1019.0</td>
      <td>NW</td>
      <td>9.84</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2010-01-01 04:00:00</th>
      <td>NaN</td>
      <td>-20</td>
      <td>-12.0</td>
      <td>1018.0</td>
      <td>NW</td>
      <td>12.97</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
pd.isnull(dataset).any()
```




    pollution     True
    dew          False
    temp         False
    press        False
    wnd_dir      False
    wnd_spd      False
    snow         False
    rain         False
    dtype: bool




```python
#缺失数据用平均值填充
dataset=dataset.fillna(dataset.mean()['pollution'],axis=1)
```


```python
dataset.wnd_dir.unique()
```




    array(['NW', 'cv', 'NE', 'SE'], dtype=object)




```python
dataset.wnd_dir=dataset.wnd_dir.map({'NW':0,'cv':1,'NE':2,'SE':3})
```

#### 保存数据清洗结果


```python
dataset.to_csv('./dataset/beijing_pm25.csv')
```

## 加载数据


```python
data_set = pd.read_csv('./dataset/beijing_pm25.csv')
series = data_set.iloc[:,1]
series_values=series.values.astype('float32')
```


```python
plt.figure(figsize=(20,12))
for i in range(1,data_set.shape[1]):
    plt.subplot(data_set.shape[1],1,i+1)
    plt.plot(data_set.values[:,i])
    plt.title(data_set.columns[i],y=0.5,loc='right')
plt.show()
```


![png](output_15_0.png)


### 相关性分析


```python
corr_all = data_set.drop('date', axis = 1).corr()

mask = np.zeros_like(corr_all, dtype = np.bool)
mask[np.triu_indices_from(mask)] = True

f, ax = plt.subplots(figsize = (10, 6))

sns.heatmap(corr_all,mask = mask,linewidths=0.25,vmax=1.0, square=True,
            cmap="YlGnBu", linecolor='black', annot=True)
plt.savefig('./Correlation_Analysis.pdf')
plt.show()
```


![png](output_17_0.png)


对于pm2.5来说没有看到比较强的相关指标  
可以看到露水和温度呈现正相关特征  
气压和露水呈现负相关特征  
气压和温度呈现负相关特征


```python

```
