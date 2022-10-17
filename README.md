# COVID-19 Analysis in Indonesia

## Importing Data

Importing the dataset with API from __[covid19.go.id](https://www.covid19.go.id)__

_Info: The url of the dataset has been updated on **June 6, 2022**_

```
library(httr)
set_config(config(ssl_verifypeer = 0L))
resp <- GET ("https://storage.googleapis.com/dqlab-dataset/update.json")
status_code(resp)
```

`status_code` shows **200**, means importing dataset request has been successfully fulfilled. Use  `headers()` to see the metadata, including last update date
```
headers(resp)

```

Then extract the data as `cov_id_raw`.


```
cov_id_raw <- content(resp, as = "parsed", simplifyVector = TRUE) 

```
</p>

## Exploration Data Analysis

The functions `length()` and `names()` are used to observe how many components there are and what are the names of the components in the `cov_id_raw` object. Then extract another component with `update` column only and save it with the name `cov_id_update`.

```
length(cov_id_raw)
names(cov_id_raw)
cov_id_update <- cov_id_raw$update
```

<img width="212" alt="image" src="https://user-images.githubusercontent.com/104981673/196123718-6c42b48f-b562-4f53-9a36-22cc4d3c7162.png">


### Analyzing the Data

For better understanding, we have to know the variables of `cov_id_update`

```
lapply(cov_id_update,names)
```

<img width="430" alt="image" src="https://user-images.githubusercontent.com/104981673/196130662-68b64648-e2a4-476a-8d72-88d7b64f2645.png">

Analyze the data obtained using these codes to answer the following questions:

```
cov_id_update$penambahan$tanggal
cov_id_update$penambahan$jumlah_sembuh
cov_id_update$penambahan$jumlah_meninggal
cov_id_update$total$jumlah_positif
cov_id_update$total$jumlah_meninggal
```


1. When is the latest update date for new case data? 
   **May 14th 2022**

2. How many new recovered cases?
   **416 cases**
   
3. What is the number of additional deaths cases?
   **5 cases**
   
4. What is the latest total number of positive cases?
   **6.050.519 cases**
   
5. What is the latest total number of deaths?
   **156.453**



### How about COVID-19 in West Java? 

 

What if you want to focus on COVID-19 data in your current province of residence?

[covid19.go.id](https://www.covid19.go.id) provides provincial-level COVID-19 case data at different API addresses. For example, data regarding COVID-19 West Java, where I currently live, is available at https://storage.googleapis.com/dqlab-dataset/prov_detail_JAWA_BARAT.json and can be accessed using the following line of code:

```
resp_jabar <- GET("https://storage.googleapis.com/dqlab-dataset/prov_detail_JAVA_WEST.json")
cov_jabar_raw <- content(resp_jabar, as = "parsed", simplifyVector = TRUE)
```

Now run the `names()` function on the `cov_jabar_raw` to find out the main element names available and answer the following questions:

1. What is the total number of COVID-19 cases in West Java? **1.105.134 cases**
2. What is the percentage of deaths from COVID-19 in West Java? **1.425619%**
3. What is the percentage of recovery rate from COVID-19 in West Java? **98.28238%**



### West Java COVID-19 Progress

The historical data on the progress of COVID-19 is stored under the name `list_perkembangan`. Data from `cov_jabar_raw` are extracted and the result are saved as an object named `cov_jabar`. After that, we can observe the `cov_jabar` structure using the `str()` and `head()` functions

```
cov_jabar <- cov_jabar_raw$list_perkembangan
str(cov_jabar)
head(cov_jabar)
```

## Data Wrangling

After extracting and observing `cov_jabar`, there are some irregularities in the data such as date column and inconsistent column writing formats. The following steps are done to make the data easier to understand, using **dplyr** library.

```
library(dplyr)
```

1. Removing the `DIRAWAT_OR_ISOLASI` and `AKUMULASI_DIRAWAT_OR_ISOLASI` columns
2. Delete all columns that contain cumulative values
3. Rename the column `KASUS` to `kasus_baru`
4. Change the writing format of the `MENINGGAL` and `SEMBUH` columns to lowercase
5. Correct the data in the date column


Use the pipe operator (%>%) to string the functions into a pipeline. Save the result with the name `new_cov_jabar`.


```
new_cov_jabar <-
  cov_jabar %>% 
  select(-contains("DIRAWAT_OR_ISOLASI")) %>% 
  select(-starts_with("AKUMULASI")) %>% 
  rename(
    kasus_baru = KASUS,
    meninggal = MENINGGAL,
    sembuh = SEMBUH
    ) %>% 
  mutate(
    tanggal = as.POSIXct(tanggal / 1000, origin = "1970-01-01"),
    tanggal = as.Date(tanggal)
  )
str(new_cov_jabar)  
```

## Data Visualization


```
library(ggplot2)
library(hrbrthemes)
```

Data visualization for this dataset is using **ggplot2** and **hrbrthemes** library, with `tanggal` and `kasus_baru` as the variable to see the progress of the increase in COVID-19 cases through graphic visualization.

```
ggplot(data = new_cov_jabar, aes(x = tanggal, y = kasus_baru)) +
  geom_col()
```
![image](https://user-images.githubusercontent.com/104981673/196190012-ef455952-dbc8-482e-9d37-1f1be36f1c35.png)

