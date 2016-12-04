---
layout: post
title: 使用R爬取HMDB和KEGG数据库
description: 因为最近需要使用代谢通路数据，因此想到了使用R编写爬虫爬取KEGG和HMDB数据。
category: blog
---

## **R语言爬虫**
******************************************
虽然相对于python来说，R语言爬虫并不是那么流行，但是对于比较小的数据爬取量，使用R还是很方便的。R的数据爬取比较流行的是利用XML和RCurl包进行爬取，在这篇博客里面，我就利用XML和RCurl包进行KEGG和HMDB的数据爬取。

## **爬取KEGG通路信息**
******************************************
因为我需要的信息是KEGG的通路信息，比较简单，也就是每个通路包含哪些代谢物，只要人的metaboloic pathway，因此，我需要先将KEGG中的通路的网页链接拿到。

```
library(XML)
library(RCurl)
##从kegg主页上抓取代谢通路的url
URL = getURL("http://www.genome.jp/kegg/pathway.html#global")
doc <- htmlParse(URL,encoding="utf-8")
xpath.a <- "//a/@href"
node <- getNodeSet(doc, xpath.a)
url1 <- sapply(node, as.character)

xpath.b <- "//a[@href]"
name <- getNodeSet(doc, xpath.b)
name <- sapply(name, xmlValue)

name2 <- name[59:247]
url2 <- url1[59:247]

url3 <- url2[grep("show", url2)]

pathwat.name <- NULL
metabolite.id <- list()
metabolite.name <- list()
for (i in 1:length(url3)) {
  cat(paste(i,"/",length(url3)))
  cat("\n")
  URL <- paste("http://www.genome.jp", url3[i], sep = "")
  URL = getURL(URL)
  doc<-htmlParse(URL,encoding="utf-8")
  xpath <- "//option[@value='hsa']"
  node<-getNodeSet(doc, xpath)
  if (length(node) ==0 ) {
    cat("No human pathwat.")
    next()
  }else{
    URL <- paste("http://www.genome.jp", url3[i], sep = "")
    URL <- gsub(pattern = "map=map", replacement = "map=hsa", x = URL)
    doc<-htmlParse(URL,encoding="utf-8")
    xpath1 <- "//title"
    node<-getNodeSet(doc, xpath1)
    pathway.name[i] <- xmlValue(node[[1]])
    pathway.name[i] <- substr(pathway.name[i], start = 2, stop = nchar(pathway.name[i])-1)

    xpath2 <- "//area[@shape='circle']/@title"
    node<-getNodeSet(doc, xpath2)
    metabolite <- lapply(node, function(x) as.character(x))
    metabolite.name[[i]] <- substr(metabolite, start = 9, nchar(metabolite)-1)
    metabolite.id[[i]] <- substr(metabolite, start = 1, stop = 6)
  }
}
```

下面对爬取到的代谢通路进行筛选。

```
idx <- which(!is.na(pathway.name))
pathway.name1 <- pathway.name[idx]
metabolite.id1 <- metabolite.id[idx]
metabolite.name1 <- metabolite.name[idx]

pathway.name2 <- pathway.name1[-c(83,84)]
metabolite.id2 <- metabolite.id1[-c(83,84)]
metabolite.name2 <- metabolite.name1[-c(83,84)]
```

将爬取到的信息保存输出。

```
met.name <- NULL
met.id <- NULL
path.name <- NULL
for(i in 1:length(pathway.name2)) {
  met.name[i] <- paste(metabolite.name2[[i]], collapse = ";")
  met.id[i] <- paste(metabolite.id2[[i]], collapse = ";")
  path.name[i] <- gsub(pattern = "KEGG PATHWAY: ", "", pathway.name2[i])
  path.name[i] <- substr(path.name[i], start = 1, stop = nchar(path.name[i])-23)
}


kegg <- data.frame(path.name, met.name, met.id)
write.csv(kegg, "kegg.csv", row.names = F)

save(path.name, file = "path.name")
save(met.name, file = "met.name")
save(met.id, file = "met.id")

kegg.met <- list()
kegg.met[[2]] <- sapply(path.name, list)
kegg.met[[1]] <- metabolite.name2
kegg.met[[3]] <- metabolite.id2

names(kegg.met) <- c("gs", "pathwaynames", "metid")

save(kegg.met, file = "kegg.met")
```


code 1: Installation of *MetCleaning*

```
##pcaMethods and impute should be installed form bioconductor
##pcaMethos
source("http://bioconductor.org/biocLite.R")
    biocLite("pcaMethods")
##impute
source("http://bioconductor.org/biocLite.R")
    biocLite("impute")
 if(!require(devtools)) {
  install.packages("devtools")
 }
 library(devtools)
 install_github("jaspershen/MetCleaning")
 library(MetCleaning)
 help(package = "MetCleaning")
```

## **Data cleaning**
******************************************
Data cleaning is integrated as a function named as *MetClean* in *MetCleaning*. We use the demo data as the example. Copy the code below and paste in you R console.

code 2: Demo data of *MetClean*

```
##demo data
data(data, package = "MetCleaning")
data(sample.information, package = "MetCleaning")
##demo work directory
dir.create("Demo for MetCleaning")
setwd("Demo for MetCleaning")
##write files
write.csv(data, "data.csv", row.names = FALSE)
write.csv(sample.information , "sample.information.csv", row.names = FALSE)
```

The demo data have been added in your work directory and organized in you work directory as Figure 2 shows. It contains two files, "data.csv" and "sample.information.csv".
1. "data.csv" is the metabolomic dataset you want to process. Rows are features and columns are feature abundance of samples and information of features. The information of features must contain "name" (feature name), "mz" (mass to change ratio) and "rt" (retention time). Other information of features are optional, for example "isotopes" and "adducts". The name of sample can contain ".", but cannot contain "-" and space. And the start of sample name cannot be number. **For example, "A210.a" and "A210a" are valid, and "210a" or "210-a" are invalid.**
2. "sample.information.csv" is sample information for metabolomic dataset. Column 1 is "sample.name" which is the names of subject and QC samples. Please confirm that the sample names in "sample.information.csv" and "data.csv" are completely same. Column 2 is "injection.order" which is the injection order of QC and subject samples. Column 3 is "class", which is used to distinguish "QC" and "Subject" samples. Column 4 is "batch" to provide acquisition batch information for samples. Column 5 is "group", which is used to label the group of subject sample, for example, "control" and "case". The "group" of QC samples is labeled as "QC".

![Figure2 Data organisation of MetCleaning](/images/metcleaning/data organisation.jpg)

Then you can run *MetClean* function to do data cleaning of data. All the arguments of *MetClean* can be found in the other functions in *MetCleaning*. You can use *help(package = "MetCleaning")* to see the help page of *MetCleaning*.

code 3: Running of *MetClean*

```
##demo data
MetClean(polarity = "positive")
```

Running results of *MetClean*
1.Missing or zero values filtering. In the missing or zero value filtering step, if there are samples which beyond the threshold you set, you should decide to filter them or not. We recommend to remove all of them as Figure 3 shows.

![Figure3 Missing or zero value filtering](/images/metcleaning/mv filter.jpg)

2.Sample filtering. In the QC or subject sample filtering step (based on PCA), if there are samples which beyond the threshold you set, you should decide to filter them or not. We don't recommend to remove them as Figure 4 shows, because they should be consired combined other information.

![Figure4 Sample filtering](/images/metcleaning/sample filter.jpg)

3.Output files. Output files of *MetClean* are listed as Figure 5 shows.
(1) "1MV overview", "2MV filter", "3Zero overview" and "4Zero filter" are missing and zero values filtering information.
(2) "5QC outlier filter" and "6Subject outlier filter" are sample filtering based on PCA information.
(3) "7Normalization result" is the data normalization information for each batch.
(4) "8Batch effect" is the batch effect both in before and after data cleaning.
(5) "9metabolite plot" is the scatter plot for each feature.
(6) "10Data overview" is the overview of data.
(7) "11RSD overview" is the RSD distribution for each batch both before and after data cleaning.
(8) **"data_after_pre.csv", "qc.info.csv" and "subject.info"** are the data and sample information after data cleaning.
(9) "intermediate" is the intermediate data during processing.

![Figure5 Output files of *MetClean*](/images/metcleaning/output files of MetClean.jpg)

## **Statistical analysis**
******************************************
Data statistical analysis is integrated as a function named as *MetStat* in *MetCleaning*. We use the demo data as the example. **Please note that now *MetStat* can only process two class data.** Copy the code below and paste in you R console.

code 4: Demo data of *MetStat*

```
data("met.data.after.pre", package = "MetCleaning")
data(new.group, package = "MetCleaning")
##create a folder for MetStat demo
dir.create("Demo for MetStat")
setwd("Demo for MetStat")
## export the demo data as csv
write.csv(new.group, "new.group.csv", row.names = FALSE)
```

The demo data have been added in your work directory. "new.group.csv" is a sample.information which has been changed the group information you want to use for statistical analysis. For the sample which you don't want to use them for statistical analysis, you can set they group information as NA like Figure 6 shows.

![Figure6 new group information](/images/metcleaning/new.group.jpg)

code 5: Running of *MetStat*

```
MetStat(MetFlowData = met.data.after.pre, new.group = TRUE)
```

Running results of *MetStat*
1.Sample removing. Firstly, you need to confirm the samples which you want to remove form dataset as Figure 7 shows.

![Figure7 sample removing confirmation](/images/metcleaning/sample remove.jpg)

2.Number of component selection in PLS-DA analysis. In PLS-DA analysis, you should manually select the best choice of the number of component. When the Console show "How many comps do you want to see?", you can type 10 and enter "Enter" key. Then a MSE plot is showing, and the best number of component is the one has the smallest CV values. So type the number (in this example is 4) and enter "Enter" key.

![Figure8 Number of component selection in PLS-DA analysis](/images/metcleaning/PLS analysis.jpg)

3.Output files. Output files of *MetStat* are listed as Figure 9 shows.
(1) "12PCA analysis" is the PCA score plot.
(2) "13PLS analysis" contains the PLS-DA results.
(3) "14heatmap" is the heatmap.
(4) "15marker selection" contains the information of markers, volcano plot and boxplots of markers.
(5) **"data_after_stat.csv", "qc.info.csv" and "subject.info"** are the data and sample information after statistical analysis.
(6) "intermediate" is the intermediate data during processing.

![Figure9 Output files of *MetStat*](/images/metcleaning/output files of MetStat.jpg)

[JasperShen]:    http://jaspershen.com  "JasperShen"
