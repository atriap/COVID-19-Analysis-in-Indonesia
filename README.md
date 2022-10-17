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

The historical data on the progress of COVID-19 is stored under the name `list_perkembangan`. Data from `cov_jabar_raw` are extracted and the results are saved as an object named `cov_jabar`. After that, we can observe the `cov_jabar` structure using the `str()` and `head()` functions

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

Data visualization for this dataset is using **ggplot2** and **hrbrthemes** package, with `tanggal` and `kasus_baru` as the variable to see the progress of the  COVID-19 cases through graphic visualization.

### Daily Positive Cases of COVID-19 in West Java

```
ggplot(new_cov_jabar, aes(tanggal, kasus_baru)) +
	geom_col(fill = "salmon") +
labs(
	x = NULL,
	y = "Jumlah kasus",
	title = "Kasus Harian Positif COVID-19 di Jawa Barat",
	subtitle = "Terjadi pelonjakan kasus di awal bulan Juli akibat klaster Secapa AD Bandung",
	caption = "Sumber data: covid19.go.id"
) +
theme_ipsum(
	base_size = 13,
	plot_title_size = 21,
	grid = "Y",
	ticks = TRUE
) +
theme(plot.title.position = "plot")
```

![image](https://user-images.githubusercontent.com/104981673/196191498-81a2fa2a-1652-4d9f-9d66-267d46daf187.png)


### Daily Recovery Cases of COVID-19 in West Java

```
ggplot(new_cov_jabar, aes(tanggal, sembuh)) +
  geom_col(fill = "olivedrab2") +
  labs(
    x = NULL,
    y = "Jumlah kasus",
    title = "Kasus Harian Sembuh Dari COVID-19 di Jawa Barat",
    caption = "Sumber data: covid.19.go.id"
  ) +
  theme_ipsum(
    base_size = 13, 
    plot_title_size = 21,
    grid = "Y",
    ticks = TRUE
  ) +
  theme(plot.title.position = "plot")
 ```

![image](https://user-images.githubusercontent.com/104981673/196193531-c457578d-81df-4774-a946-42b53741c8cd.png)


### Daily Deaths Cases of COVID-19 in West Java

```
ggplot(new_cov_jabar, aes(tanggal, meninggal)) +
  geom_col(fill ="darkslategray4") +
  labs(
    x = NULL,
    y = "Jumlah kasus",
    title = "Kasus Harian Meninggal Akibat COVID-19 di Jawa Barat",
    caption = "Sumber data: covid19.go.id"
  ) +
  theme_ipsum(
    base_size = 13, 
    plot_title_size = 21,
    grid = "Y",
    ticks = TRUE
  ) +
  theme(plot.title.position = "plot")
```
   
![image](https://user-images.githubusercontent.com/104981673/196193807-d1148ff1-dedb-4889-9179-ed3fda5096f9.png)
 

The graphs shows there is a daily fluctuation in the increase in cases. Based on this you then want to try to observe how are the progresses in a span of weeks. How to do it?

The `week()` function from **lubridate** package will be used to extract week information in a year as week.

```
library(lubridate)
```

New object will be saved as `cov_jabar_pekanan`. To perform data inspections,  `glimpse()` function of **dplyr** is used.

```
cov_jabar_pekanan <- new_cov_jabar %>% 
  count(
    tahun = year(tanggal),
    pekan_ke = week(tanggal),
    wt = kasus_baru,
    name = "jumlah"
  )

glimpse(cov_jabar_pekanan)
```
