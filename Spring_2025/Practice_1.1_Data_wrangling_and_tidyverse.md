
## Data import and reshaping data
### Importing Spreadsheets

#### Here we introduce the `tidyverse` packages `readr` and `readxl`, which facilitate importing data from several formats of spreadsheet files.
One of the initial strugles in R is how to allow R to access your data, so you can manipulate it. 
It is the same idea as opening a file in a program like *Word* or *Excel*. The difference here is that in R you need a command for opening or loading your data.

```
library(tidyverse)
```

### Paths and the Working Directory
One simple way to handle datasets is to set your *working directory* to the same folder as the data.
In our case, we can set it to the **Documents** folder. As usually it is an easy to find folder.
It is not a good practice, but for now, it will be a soft start. 
In the future we will create a folder with the purpose of storing all the information necessary to process our data.

```
# This command will tell you what is your current working directory
getwd()
# If it is not the "Documents" folder, you will need to find the address of your "Documents" folder and add it to the command setwd(dir = "path_of_your_Documents_folder")

# Check if you correcly entered the folder
getwd()

# Create a variable that stores your working directory address
path <- getwd()
# For me this would be the same as
path <- "C:/Users/dmo042/OneDrive - UiT Office 365/Documents"

# This next command creates a folder:
dir.create("Datasets")

# And with this command you can enter the folder you just created:
setwd("Datasets/")

# Check what are the files in your working directory
list.files(path)

# Add the name of the file we will use to an object
filename <- "GFW-fishing-vessels-v2.csv"

# Store the full path of our file
fullpath <- file.path(path, filename)
fullpath

# Copy files between folders
file.copy(fullpath, getwd())

list.files(getwd())
list.files()
```
Check if the file is now in your working directory using the `file.exists()` 
```
# function:
file.exists(filename)
```
---
### The `readr` and `readxl` packages
#### `readr` is the `tidyverse` library that includes functions for reading data stored in text file spreadsheets into R. 
#### The following functions are available to read-in spreadsheets:

| Function | Format | Typical suffix |
|----------|--------|---| 
| read_table() | white space separated values | .txt |
| read_csv() | comma separated values|  .csv |
| read_csv2() | semicolon separated values | .csv |
| read_tsv() | tab delimited separated values | .tsv |
| read_delim() | general text file format, must define delimiter | .txt |

####  The `readxl` package provides functions to read in Microsoft Excel formats:

| Function | Format | Typical suffix |
|----------|--------|---| 
| read_excel() | auto detect the format | .xls, .xlsx|
| read_xls() | original format |  .xls |
| read_xlsx() | new format | .xlsx |
 
Note that the Microsoft Excel formats permit you to have more than one spreadsheet in one file.
These are referred to as _sheets_. The functions above read the first sheet by default.  
The `excel_sheets()` function gives us the names of the sheets in an excel file.
These names can then be passed to the `sheet` argument in the three functions above to read sheets other than the first.

```
# Read the first six lines
read_lines(filename, n_max = 6)
# Read the full file
dat <- read_csv(filename)
# Note that dat is a tibble
head(dat)
class(dat)
str(dat)
```
---
### R-base functions
#### R-base also provides import functions. These have similar names to those in the `tidyverse`:
#### `read.table`, `read.csv` and `read.delim` for example. There are a couple of important differences. 

```
# To show this we read the data with an R-base function:
dat2 <- read.csv(filename)
head(dat2)
class(dat2)
str(dat2)
```

#### Downloading files
Another common place for data to reside is on the internet. 
We can download these files and then import them or even read them directly from the web.  
For example, we are going to download the file GFW-fishing-vessels-v2.csv from our GitHub repository.  

The file has the following url:
```
url <- "https://raw.githubusercontent.com/bioinfo-arctic/FSK2053/main/Spring_2024/data/GFW-fishing-vessels-v2.csv"

```

The `read_csv()` function can read these files directly:

```
dat <- read_csv(url)
```

If you want to have a local copy of the file, you can use `download.file()`. 

```
download.file(url, "local-GFW-fishing-vessels-v2.csv")
list.files()
```

---
### Nuances
When reading in spreadsheets many things can go wrong. 
The file might have a multiline header, missing cells, or it might use an unexpected [encoding](https://en.wikipedia.org/wiki/Character_encoding)  
We recommend you read this [post](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/). 
With experience you will learn how to deal with different challenges.  
Carefully reading the `help()` files for the functions discussed here will help.  
Two other functions that are helpful are `scan()` and `readLines()`.  
With `scan()` you can read in each cell of a file. Here is an example:

```
x <- scan(url, sep=",", what = "c")
x[1:60]
```

#### Removing a file
Now that we are done with the example, we will remove the example file we copied over to our working directory using the function `file.remove()`.

```
file.remove("local-GFW-fishing-vessels-v2.csv")
```
---
### Tidy data

```
library(tidyverse)
names(dat)
years <- as.character(paste0("fishing_hours_",2012:2020))
```

We will filter our results for just four countries and then we will select just a smaller set of variables

```
wide_data <- dat %>% 
  filter(flag_gfw %in% c("NOR","GBR","ESP","CHN")) %>%
  select(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw,all_of(years))
head(wide_data)
```

We can represent the fishing hours per vessel for 2020

```
wide_data %>% ggplot(aes(tonnage_gt_gfw, fishing_hours_2020, color = flag_gfw)) +
  geom_point()
```

However, if we want to represent several years in the plot, we cannot do that directly. Because the table is in **wide_format** . We have to change it to **tidy_data**.

There are two important differences between the wide and tidy formats.  
First, in the **wide_format**, each row includes several observations. 
Second, one of the variables, year, is stored in the header.
The `ggplot` code we introduced earlier no longer works for all the years. 
So to use the __tidyverse__ library we need to wrangle this data into __tidy__ format.

## Reshaping data
Having data in `tidy` format is what makes the `tidyverse` flow. 
After the first step in the data analysis process, importing data, a common next step is to reshape the data into a form that facilitates the rest of the analysis. The `tidyr` package includes several functions that are useful for tidying data. 

---
####  The `gather()` function

One of the most used functions in this package is `gather()`, which converts wide 
data into tidy data.

Let's see a simple example with our data. Here we have the fishing hours data in wide format:
```
head(wide_data)
```
To use `gather()`, we need to specify the names of the new columns that will be created with the arguments
_variable_ and _value_. The third argument will be the names of the columns that will be gathered 
into the tidy format.
```
tidy_data <- wide_data %>% gather("variable","value",all_of(years))
tidy_data
```
The first argument sets the column/variable name that will hold the variable that
is currently kept in the wide data column names. We can name it anything. The second argument sets the column/variable name that will hold the values in the column cells.

#### A more practical solution would be to call the columns `year` and `fishing hours` like this:
```
tidy_data <- wide_data %>% gather("year","fishing_hours",all_of(years))
tidy_data
```
Finally, we would like to change the contents of the cells for year to the "numeric year".
We can use the function `gsub()` which will replace (substitute) a string for another one.

#### Note that the result is still a character string
```
tidy_data$year <- gsub("fishing_hours_","",tidy_data$year)
class(tidy_data$year)
```
We can represent the plot for all years together
```
tidy_data %>% ggplot(aes(tonnage_gt_gfw, fishing_hours, color = flag_gfw)) +
  geom_point()
```
And we can use different shapes for every year
```
tidy_data %>% ggplot(aes(tonnage_gt_gfw, fishing_hours, color = flag_gfw)) +
  geom_point(aes(shape = year))
```
Since the shape palette is not good for more than 6 values, we will change colors and shape
```
tidy_data %>% ggplot(aes(tonnage_gt_gfw, fishing_hours, color = year)) +
  geom_point(aes(shape = flag_gfw))
```
Another way to write this code is to specify which columns will **not** be gathered rather than all the columns that will be gathered:
```
tidy_data <- wide_data %>% gather("year","fishing_hours",-c(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw))
```
---
#### The functions `mutate()`  and `transmute()`
We can create new columns or overwrite existing ones using the function `mutate()`.
The existing columns will be preserved.
```
tidy_data <- wide_data %>% 
  gather("year_string","fishing_hours",-c(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw)) %>% 
  mutate(year = gsub("fishing_hours_","",year_string))
```
If we use the same name, the column will be overwritten.
```
tidy_data <- wide_data %>% 
  gather("year","fishing_hours",-c(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw)) %>% 
  mutate(year = gsub("fishing_hours_","",year))
```
We can use the function `transmute()` to create a whole new _tibble_ with new variables.
The existing variables will be removed:
```
tidy_data2 <- wide_data %>% 
  gather("year","fishing_hours",-c(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw)) %>% 
  transmute(tonnage = as.integer(tonnage_gt_gfw),year = gsub("fishing_hours_","",year), 
  hours = fishing_hours)
```
Note that we have a lot of observations containing _NA_ values. 
We can remove the observations with _NA_ using `drop_na()`
```
tidy_data <- wide_data %>% 
  gather("year","fishing_hours",-c(mmsi,flag_gfw,vessel_class_gfw,tonnage_gt_gfw)) %>% 
  mutate(year = gsub("fishing_hours_","",year)) %>% 
  drop_na()
```
---
#### The  `spread()` function
It is sometimes useful for data wrangling purposes to convert tidy data into wide data. 
We often use this as an intermediate step in tidying up data. 
The `spread()` function is basically the inverse of `gather()`. 
 - The first argument tells `spread` which variable will be used as the column names. 
 - The second argument specifies which variable to use to fill out the cells:
```
new_wide_data <- tidy_data %>% spread(year, fishing_hours)
new_wide_data
```
Note that _NA_ values are added when there are missing values for a given vessel

This is useful, for example, if we want to select only the vessels that worked during all the years in the record
```
new_wide_data_all <- tidy_data %>% 
	spread(year, fishing_hours) %>% 
	drop_na()
```
---
#### The functions `pivot_longer()` and `pivot_wider()`
`pivot_longer()` is another way to produce a tidy **longer** dataset from a wide data table.
Remember that we used the function `gather()` like this:
```
tidy_data <- wide_data %>% gather("year","fishing_hours",all_of(years))  %>% 
  mutate(year = gsub("fishing_hours_","",year))
```
We can get the same result using `pivot_longer()`.
We need three parameters:
- The set of columns whose names are values, not variables. In this example, those columns are `all_of(years)`.
- The name of the variable to move the column names to. Here it is "year".
- The name of the variable to move the column values to. Here it’s "fishing_hours".

```
tidy_data <- wide_data %>%
	pivot_longer(all_of(years),names_to = "year", values_to = "fishing_hours") %>% 
	drop_na()
```
We add the `mutate()` function for changing the year column
```
tidy_data <- wide_data %>% 
	pivot_longer(all_of(years),names_to = "year", values_to = "fishing_hours") %>% 
	mutate(year = gsub("fishing_hours_","",year)) %>% 
	drop_na()
```
Also, we can use `pivot_wider()` to get a wide data_frame from the tidy data_frame.
Now we need just two parameters:
 - The column to take variable names from. Here, it’s year.
 - The column to take values from. Here it’s fishing_hours.
```
new_wide_data <- tidy_data %>% 
	pivot_wider(names_from = year, values_from=fishing_hours)
```
The function `pivot_longer()` is the replacement for `gather()`and `pivot_wider()` is the replacement for `spread()`.

Both are designed to be simpler and can handle more cases than `gather()` and `spread()`.
The **tidyverse** developers highly recommends to use the new functions, although `gather()` and `spread()` are not going away but will not be actively developed.

---

#### The function `separate()` 

The data wrangling shown above was simple compared to what it is usually required. This is because the data came from a structured table source.
Here is another example that is slightly more complicated. It includes two variables: life expectancy as well as fertility. 
However, the way they are stored is not tidy and, as we will explain, not optimal.
We will download it from our Github repository:
```
url <- "https://raw.githubusercontent.com/DataScienceFishAquac/FSK2053-2021/main/datasets/fertility-life_expectancy.csv"
```
The `read_csv()` file can read these files directly:
```
raw_dat <- read_csv(url)
head(raw_dat)
```
First note that the data is in wide format. Second, note that now there are values for two variables with the column names encoding which column represents which variable. 
We can start the data wrangling with the `gather()` function, We will call the variable `key`, the default:
```
dat_fert <- raw_dat %>% 
	gather(key, value, -country)
head(dat_fert)  
```
The result is not exactly what we refer to as tidy, since each observation is associated with two rows instead of one.
We want to have the values from the two variables, fertility and life expectancy, in two separate columns.
The first challenge to achieve this is to separate the `key` column into the year and the variable type. 
Note that the entries in this column separate the year from the variable name with an underscore: 
```
dat_fert$key[1:5]
```
Encoding multiple variables in a column name is such a common problem that the `readr()` package includes a function to separate these columns into two or more. 
The `separate()` function takes 3 arguments: 

1. The name of the column to be separated
2. The names to be used for the new columns 
3. The character that separates the variables

So a first attempt at this is:
```
dat_new <- dat_fert %>% 
	separate(key, c("year", "variable_name"), "_")
```
Actually, because "_" is the default separator we can simply write:
```
dat_new <- dat_fert %>% 
	separate(key, c("year", "variable_name"))
```
However, we run into a problem. Note that we receive the warning:
```
Expected 2 pieces. Additional pieces discarded in 112 rows:
```
and the `life_expectancy` variable is truncated to `life`. 
This is because the `_` is used to separate `life` and `expectancy` not just year and variable name. 
We could add a third column to catch this and let the `separate` function know which column to fill in with _NA_, when there is no third value.
Here we tell it to fill the column on the right:
```
dat_new <- dat_fert %>% 
	separate(key, c("year", "first_variable_name", "second_variable_name"), fill = "right")
```
However, if we read the `separate` help file we find that a better approach is to merge the last two variables when there is an extra separation:
```
dat_new <- dat_fert %>% 
	separate(key, c("year", "variable_name"), sep = "_", extra = "merge")
dat_new
```
This achieves the separation we wanted. The dataset is almost in tidy format. 
 But the column 'value' is merging two variables. This is not optimal, for example, for plotting purposes.
So we want to create a column for each variable. 
We can use the `spread` function:
```
dat_new <- dat_fert %>%
	separate(key, c("year", "variable_name"), sep = "_", extra = "merge") %>%
	spread(variable_name, value) 
dat_new
```
The data is now in tidy format with one row for each observation with three variables: `year`, `fertility` and `life expectancy`.

---

#### Now we can plot the data:
```
dat_new %>% ggplot(aes(x=year),color = country,) +
  geom_line(aes(y = life_expectancy, group=country, colour = country))  +
  geom_line(aes(y = fertility/.1, group=country,colour = country, 
linetype="fertility"),linetype="dashed") +
  scale_y_continuous(sec.axis = sec_axis(~.*.1, name = "fertility")) +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```
---
#### The function`unite()`
It is sometimes useful to do the inverse of `separate`, i.e. unite two columns into one. 
So, although this is *not* an optimal approach, had we used this command to separate: 
```
dat_new <- dat_fert %>% 
	separate(key, c("year", "first_variable_name", "second_variable_name"), fill = "right")
dat_new
```
We can achieve the same final result by uniting the second and third column like this:
```
dat_new <- dat_fert  %>% 
  separate(key, c("year", "first_variable_name", "second_variable_name"), fill = "right") %>%
  unite(variable_name, first_variable_name, second_variable_name, sep = "_")
dat_new
```
Then spreading the columns:
```
dat_new <- dat_fert   %>% 
  separate(key, c("year", "first_variable_name", "second_variable_name"), fill = "right") %>%
  unite(variable_name, first_variable_name, second_variable_name, sep = "_") %>%
  spread(variable_name, value) %>%
  rename(fertility = fertility_NA)
dat_new
```
