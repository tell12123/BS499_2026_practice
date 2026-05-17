# Week11 Data Processing Practice

This week practices how to prepare mixed clinical data for clustering analysis in R. The main goal is to make different types of patient variables more comparable before clustering.

Clinical datasets often contain many different variable types. For example, some variables are categorical, such as disease presence or severity group, while other variables are continuous, such as age, BMI, white blood cell count, or laboratory values. If these values are used directly, variables with large numeric ranges can dominate the clustering result. Therefore, this practice transforms selected continuous or ordinal variables into values between 0 and 1 using a logistic transformation.

---

## 1. Read clinical data into R

```r
file_data <- read.table("CHIP_COVID-19_210405_220311_origianl_data.txt", header = T)
```

This line reads a text file into R and saves it as an object named `file_data`.

`read.table()` is a basic R function used to read table-like data files. The first argument is the file name. In this example, the file name is `CHIP_COVID-19_210405_220311_origianl_data.txt`. This file must be in the current working directory, or R will not be able to find it.

`header = T` means that the first row of the file contains column names. In R, `T` means `TRUE`. Therefore, R will treat the first row as variable names instead of data values.

After this line is executed, `file_data` becomes a data frame. A data frame is one of the most common data structures in R. It is similar to an Excel table, where rows usually represent samples or patients and columns represent variables.

---

## 2. Set patient ID as row names

```r
row.names(file_data) <- file_data[, "ID"]
```

This line sets the row names of `file_data` using the `ID` column.

`row.names(file_data)` means the names assigned to each row of the data frame. In this dataset, each row corresponds to one patient, so it is useful to use patient ID as the row name.

`file_data[, "ID"]` means select the `ID` column from `file_data`. The comma inside `[ , ]` separates rows and columns. The part before the comma is for rows, and the part after the comma is for columns. Because the row part is empty, R selects all rows. Because the column part is `"ID"`, R selects only the `ID` column.

This step is useful because later, even after filtering or transforming the table, each patient can still be identified by their row name.

---

## 3. Define a logistic transformation function

```r
logit <- function(file_fuse, name){
  data <- file_fuse[, name]
  data <- data - median(data)
  data <- data * (2 / quantile(data)[4])
  return(1 / (1 + exp(-data)))
}
```

This code defines a function named `logit`.

A function in R is a reusable block of code. Instead of writing the same transformation many times for many columns, we define one function and apply it repeatedly.

The function has two inputs: `file_fuse` and `name`. Here, `file_fuse` is the data frame, and `name` is the column name that we want to transform.

```r
data <- file_fuse[, name]
```

This line selects one column from the data frame. For example, if `name` is `"BMI"`, this line extracts the BMI values from `file_fuse` and saves them as `data`.

```r
data <- data - median(data)
```

This line centers the data around the median. The median is the middle value of the data. By subtracting the median, values near the median become close to 0. Values larger than the median become positive, and values smaller than the median become negative.

This is useful because the logistic function changes most strongly around 0. Therefore, centering the data helps place the middle of the distribution near the most sensitive part of the logistic curve.

```r
data <- data * (2 / quantile(data)[4])
```

This line rescales the data. `quantile(data)` returns several summary values of the distribution. In R, `quantile(data)[4]` usually corresponds to the 75th percentile, also called the third quartile.

Multiplying by `(2 / quantile(data)[4])` adjusts the scale of the data so that the transformed values are not too compressed or too extreme. This helps variables with different numeric ranges become more comparable.

```r
return(1 / (1 + exp(-data)))
```

This line applies the logistic function. The logistic function converts numeric values into values between 0 and 1.

Large positive values become close to 1. Large negative values become close to 0. Values near 0 become close to 0.5.

This is useful for clustering because many clinical variables have different scales. For example, BMI may range from 10 to 40, while WBC count may range from 1000 to 10000. After logistic transformation, both variables are converted into a similar 0 to 1 range.

---

## 4. Check column names

```r
print(colnames(file_data))
```

This line prints the column names of `file_data`.

`colnames(file_data)` returns the names of all columns in the data frame. `print()` shows the result in the R console.

This step is important before selecting columns because the column names must match exactly. R is case-sensitive, so `BMI`, `bmi`, and `Bmi` are treated as different names.

If a column name is misspelled, the later code may produce an error. Therefore, checking column names before running transformation is a good habit.

---

## 5. Select variables for transformation

```r
vec <- c("Ordinal_scale", "Age", "BMI", "WBC_peak", "Lympho_lowest", "Mono_peak", "CRP_peak", "PLT_peak", "Hb_worst", "Protein_peak", "Alb_peak", "BUN_peak", "Cr_peak", "CXRmax")
```

This line creates a vector named `vec`.

In R, `c()` combines several values into one vector. Here, the vector contains column names that will be transformed by the `logit` function.

These variables are selected because they are ordinal or continuous variables. They can have different numeric ranges, so logistic transformation helps make them more comparable for clustering.

For example, `Age`, `BMI`, and `WBC_peak` are not measured on the same scale. If they are directly used for clustering, one variable may have more influence simply because it has larger numbers. The transformation reduces this scale problem.

---

## 6. Apply logistic transformation to selected columns

```r
for(i in vec){
  file_data[, i] <- logit(file_data, i)
}
```

This code applies the logistic transformation to each column listed in `vec`.

`for(i in vec)` means repeat the code inside `{ }` for every element in `vec`. During each repeat, `i` becomes one column name.

For example, in the first loop, `i` may be `"Ordinal_scale"`. In the next loop, `i` may be `"Age"`. This continues until all column names in `vec` are used.

```r
file_data[, i] <- logit(file_data, i)
```

This line replaces the original column with the transformed values.

The left side, `file_data[, i]`, selects one column in `file_data`. The right side, `logit(file_data, i)`, calculates the transformed values for that column. The assignment operator `<-` saves the transformed values back into the same column.

After this loop is finished, all selected columns in `file_data` are transformed into values between 0 and 1.

---

## 7. Reorder columns

```r
file_data <- file_data[c(1, 3, 2, 4:22)]
print(head(file_data))
```

This code changes the column order of `file_data`.

`file_data[c(1, 3, 2, 4:22)]` selects columns in a new order. This means the first column stays first, the third column moves to the second position, the second column moves to the third position, and columns 4 to 22 follow after that.

Column reordering is not always required for analysis, but it can make the table easier to read or match the expected input format for later analysis.

```r
print(head(file_data))
```

This line shows the first few rows of the data frame. `head()` is useful for quickly checking whether the data looks correct after transformation or reordering.

For beginners, it is useful to run `head(file_data)` often after important steps. This helps detect mistakes early.

---

## 8. Remove Gender column

```r
file_data <- file_data[, -grep("Gender", colnames(file_data))]
```

This line removes columns that contain the word `Gender` in their column names.

`colnames(file_data)` returns all column names. `grep("Gender", colnames(file_data))` finds the position of columns whose names contain `Gender`.

The minus sign `-` means remove those columns instead of selecting them.

Therefore, this line keeps all columns except the Gender-related column.

This step is used because Gender is removed for the subsequent analysis in this practice. In real analysis, whether to remove a variable depends on the research question and analysis design.

---

## 9. Save transformed data

```r
write.table(file_data, "CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender.txt", quote = F, sep = "\t", row.names = F)
```

This line saves the processed data frame as a text file.

`write.table()` is used to export a data frame from R.

The first argument, `file_data`, is the data frame to save.

The second argument, `"CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender.txt"`, is the output file name.

`quote = F` means do not add quotation marks around text values. In R, `F` means `FALSE`.

`sep = "\t"` means columns are separated by tab characters. This creates a tab-delimited text file, which is commonly used in bioinformatics.

`row.names = F` means row names are not written to the output file. If you want to keep patient IDs in the output, make sure the ID column still exists as a normal column before saving.

After this line is executed, a new transformed data file will be created in the current working directory.

---

# Full R code

```r
file_data <- read.table("CHIP_COVID-19_210405_220311_origianl_data.txt", header = T)

row.names(file_data) <- file_data[, "ID"]

logit <- function(file_fuse, name){
  data <- file_fuse[, name]
  data <- data - median(data)
  data <- data * (2 / quantile(data)[4])
  return(1 / (1 + exp(-data)))
}

print(colnames(file_data))

vec <- c("Ordinal_scale", "Age", "BMI", "WBC_peak", "Lympho_lowest", "Mono_peak", "CRP_peak", "PLT_peak", "Hb_worst", "Protein_peak", "Alb_peak", "BUN_peak", "Cr_peak", "CXRmax")

for(i in vec){
  file_data[, i] <- logit(file_data, i)
}

file_data <- file_data[c(1, 3, 2, 4:22)]
print(head(file_data))

file_data <- file_data[, -grep("Gender", colnames(file_data))]

write.table(file_data, "CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender.txt", quote = F, sep = "\t", row.names = F)
```

---

# Summary

This practice shows how to prepare clinical data before clustering. The key idea is that different variables can have different types and scales. Logistic transformation converts selected variables into a similar 0 to 1 range. This makes the data more suitable for clustering because the clustering result becomes less dominated by variables with large numeric ranges.
