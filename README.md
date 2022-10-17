# COVID-19 Analysis in Indonesia


Importing the dataset with API from __[covid19.go.id](https://www.covid19.go.id)__

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

## Data Exploration

The functions `length()` and `names()` are used to observe how many components there are and what are the names of the components in the `cov_id_raw` object. Then extract another component with `update` column only and save it with the name `cov_id_update`.

```
length(cov_id_raw)
names(cov_id_raw)
cov_id_update <- cov_id_raw$update
```

<img width="212" alt="image" src="https://user-images.githubusercontent.com/104981673/196123718-6c42b48f-b562-4f53-9a36-22cc4d3c7162.png">

## Analyzing the Data

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

Info: The url of the dataset has been updated on June 6, 2022, which is https://storage.googleapis.com/dqlab-dataset/prov_detail_JAVA_BARAT.json.
 

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
