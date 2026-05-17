# Week12

## Goal of this practice

This week practices:

- Distance calculation between patients
- PCA
- t-SNE
- Clustering
- Visualization of clustering results

Using transformed COVID-19 patient data. :contentReference[oaicite:0]{index=0}

---

# Load transformed data into R

```r
file<-read.table("CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender.txt", header=T)

head(file)
```

Explanation:

```r
read.table()
```

Read txt table into R.

```r
header=T
```

Means first row is column names.

```r
head(file)
```

Show first 6 rows of the data.

Purpose:
- Check data loaded correctly
- Check column names and values

---

# Load required packages

```r
library(FactoMineR)
library(ggplot2)
library(Rtsne)
library(M3C)
```

Explanation:

- `FactoMineR` → PCA analysis
- `ggplot2` → visualization
- `Rtsne` → t-SNE calculation
- `M3C` → clustering

---

# Define Euclidean distance function

```r
euclidean_distance <- function(a, b){

  return(sqrt(sum((a - b)^2)))

}
```

Explanation:

This function calculates distance between two patients.

Euclidean distance:

\[
\sqrt{(a_1-b_1)^2 + (a_2-b_2)^2 + ...}
\]

Smaller distance:
- patients more similar

Larger distance:
- patients more different

---

# Read data again

```r
file<-read.table("CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender.txt", header=T)
```

---

# Set row names

```r
rownames(file)<-file[,"ID"]
```

Explanation:

Use patient ID as row name.

Example:

Before:

| row | ID |
|---|---|
|1|101|

After:

| rowname |
|---|
|101|

Useful for matching samples later.

---

# Remove unnecessary columns and transpose matrix

```r
file_transpose<-t(file[,-grep("ID|CHIP_allns|cluster",colnames(file))])
```

Explanation:

```r
grep()
```

Find columns containing specific words.

```r
-grep()
```

Remove matched columns.

Removed:
- ID
- CHIP_allns
- cluster

Reason:
- these are labels
- not numeric features for clustering

```r
t()
```

Transpose matrix.

Rows ↔ columns.

Needed because:
- patients should become columns for distance calculation

---

# Create empty matrix for distances

```r
Euc_dist<-matrix(0,nrow(file),nrow(file))
```

Explanation:

Create empty matrix filled with 0.

Purpose:
- save patient-to-patient distances

Example:

| |P1|P2|
|---|---|---|
|P1|0| |
|P2| |0|

---

# Add row and column names

```r
colnames(Euc_dist)<-colnames(file_transpose)
rownames(Euc_dist)<-colnames(file_transpose)
```

Explanation:

Set patient names for matrix rows and columns.

---

# Calculate distances between all patients

```r
for(i in c(1:ncol(Euc_dist))){

  for(j in c(1:nrow(Euc_dist))){

    Euc_dist[i, j]<-euclidean_distance(file_transpose[,i],file_transpose[,j])

  }

}
```

Explanation:

Double for-loop.

Compare:
- patient i
- patient j

Calculate Euclidean distance and save into matrix.

Result:
- full patient distance matrix

---

# Fix random seed

```r
set.seed(1)
```

Explanation:

Fix random number generation.

Important because:
- t-SNE uses randomness
- clustering can slightly change every run

Using seed:
- results become reproducible

---

# Correlation and PCA

```r
Euc_dist_corr=cor(Euc_dist)

Whole.pca=PCA(t(Euc_dist_corr), graph=F, ncp=10)
```

Explanation:

```r
cor()
```

Calculate correlation between patients.

```r
PCA()
```

Principal Component Analysis.

Reduce dimension while preserving major variation.

```r
ncp=10
```

Use first 10 principal components.

Purpose:
- simplify high-dimensional data

---

# Create patient name list

```r
name_list=paste0("ID",row.names(Euc_dist_corr))
```

Explanation:

Add "ID" in front of row names.

Example:

```r
101 -> ID101
```

---

# Run t-SNE

```r
b=Rtsne::Rtsne(Whole.pca$var$coord,
               perplexity=10,
               check_duplicates = FALSE)
```

Explanation:

t-SNE:
- dimensionality reduction method
- visualize complex data in 2D

```r
perplexity=10
```

Important parameter.

Smaller:
- more local structure

Larger:
- more global structure

Try:
- 5
- 10
- 20
- 30

and compare results. :contentReference[oaicite:1]{index=1}

---

# Extract t-SNE coordinates

```r
YY=b$Y

rownames(YY)<-name_list

colnames(YY)<-c("feature 1", "feature 2")
```

Explanation:

Extract:
- t-SNE X coordinate
- t-SNE Y coordinate

Each patient now has:
- feature 1
- feature 2

for visualization.

---

# Run clustering

```r
m3c_clust2=M3C(t(YY))
```

Explanation:

Run clustering using M3C package.

Purpose:
- identify patient groups
- detect enriched clusters

---

# Save clustering result

```r
saveRDS(m3c_clust2,
"CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender_Euclid_dist_cor_PCA_per_10_tsne_m3c.rds")
```

Explanation:

Save R object into `.rds` file.

Useful because:
- no need to rerun all calculations later

---

# Read clustering result

```r
mat_tsne<-readRDS("CHIP_COVID-19-patients_smoothen_2_Logit_wo_gender_Euclid_dist_cor_PCA_per_10_tsne_m3c.rds")
```

Explanation:

Load saved R object back into R.

---

# Read original table again

```r
table<-read.table("CHIP_COVID-19_210405_220311_origianl_data.txt",
                  header=T,
                  row.names="ID")
```

Purpose:
- retrieve original CHIP information

---

# Create plot matrix

```r
name_list=paste0("ID",row.names(table))

row.names(table)<-name_list
```

Add patient IDs again.

---

# Combine information

```r
mat<-t(mat_tsne$realdataresults[[8]]$ordered_data)

mat_mat<-cbind(
mat[name_list,],
mat_tsne$realdataresults[[8]]$ordered_annotation[name_list,],
table[name_list,"CHIP_allns"]
)
```

Explanation:

Combine:
- t-SNE coordinates
- cluster labels
- CHIP information

into one matrix.

---

# Convert into dataframe

```r
mat_mat_d<-as.data.frame(mat_mat)

colnames(mat_mat_d)<-c("V1","V2","V3","V4")
```

Explanation:

Convert matrix into dataframe for ggplot.

Columns:
- V1 → t-SNE x
- V2 → t-SNE y
- V3 → cluster
- V4 → CHIP presence/absence

---

# Draw t-SNE plot

```r
pdf("Plot.pdf")

ggplot(mat_mat_d,
       aes(x=as.numeric(V1),
           y=as.numeric(V2),
           color=as.factor(V3),
           shape=as.factor(V4)))+

geom_point()+

scale_shape_manual(values=c(16,17))+

scale_colour_manual(values=c(
"#F27A2A",
"#F7E863",
"#69E28B",
"#ADEF59",
"#6CD7EF",
"#2A8EF2",
"#7BEFD3",
"#535CF9"))

dev.off()
```

Explanation:

```r
ggplot()
```

Make plot.

```r
color=as.factor(V3)
```

Different colors for clusters.

```r
shape=as.factor(V4)
```

Different shapes for CHIP status.

```r
pdf()
```

Save figure as PDF.

```r
dev.off()
```

Finish saving figure.

---

# Practice goals

## 1. Change logistic transformation strength

Try changing:

```r
data<-data*(2/quantile(data)[4])
```

Questions:
- Does clustering become stronger?
- Do clusters disappear?
- Does CHIP-enriched cluster remain?

:contentReference[oaicite:2]{index=2}

---

# 2. Change t-SNE perplexity

Try:

```r
perplexity=5
perplexity=10
perplexity=20
perplexity=30
```

Questions:
- Do clusters merge?
- Do clusters split?
- Does patient grouping change?

:contentReference[oaicite:3]{index=3}
