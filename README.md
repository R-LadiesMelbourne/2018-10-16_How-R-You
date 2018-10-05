How R you? - R-Ladies Melbourne code and tips!
================
01/10/2018

-   [Using `pheatmap` to visualise the structure of your data](#using-pheatmap-to-visualise-the-structure-of-your-data)
    -   [See the final result!](#see-the-final-result)
    -   [Useful links to know more about heatmaps in R](#useful-links-to-know-more-about-heatmaps-in-r)
-   [Violin plots with overlayed boxplots and coloured by group](#violin-plots-with-overlayed-boxplots-and-coloured-by-group)
-   [Matching strings](#matching-strings)

<img src="How_R_You_R-LadiesMelbourne_code_and_tips_files/figure-markdown_github/unnamed-chunk-1-1.png" style="display: block; margin: auto;" />

Using `pheatmap` to visualise the structure of your data
========================================================

**Author**: [Anna Quaglieri](https://github.com/annaquaglieri16)

I am doing my PhD in Bioinformatics and in particular I work with gene expression data from Leukemia patients belonging to several Australian clinical trials. Patients data tends to arrive in several batches since patient's genes are sequenced usually at different times. Sample also come in replicates (same sample processed twice). Even though every patient will have a lot of genes, my datasets are never too big in terms of the number of patients. When I had my first dataset, to help me reasoning and to always have the chance to look at a summarised structure of my data I came up with a way to put an excel spreadsheet with patients' information and replicates into an image with an `heatmap`. I realised that also my collaborators found this really useful and they usually have always at hand the image of the structure of the data whenever we meet. Of course, this method is not really feasable with datasets that have many samples but for me it has never been the case!

**Example data**

Below I provided a sample `excel` spreadhseet `DatasetStructure.xlsx` with a fake dataset and the script that I would use to visualise it. Of course, every patients can have many many more variables and I try to choose the most relevant ones to revent the visualisation from becoming a real mess of colours and labels (which sometimes it does anyway!)!

In the example below I want to visualise the structure of a dataset constituted by 38 patients with disease `A` or `B` for which samples were collected at 3 different time points (`Time0`, `Time1` and `Time2`).

-   Every patient (`Patient` column) has different samples extracted at different time points that I called `Time0`, `Time1` and `Time2`
-   Samples were extracted across two different batches one year apart from each other: `Batch1` and `Batch2`
-   There can be replicated samples (the same sample processed twice or more) within or between each batch
-   Patients can have one out of two disease types: `A` or `B`

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
    ## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(readxl)
library(pheatmap)

patients_infos <- read_excel(file.path("How_R_You_R-LadiesMelbourne_code_and_tips_data/DatasetStructure.xlsx")) 
head(patients_infos)
```

    ## # A tibble: 6 x 6
    ##   Patient    ID Batch  DiseaseType Gender Time 
    ##   <chr>   <dbl> <chr>  <chr>       <chr>  <chr>
    ## 1 A1        126 Batch1 A           Female Time0
    ## 2 A1        128 Batch1 A           Female Time2
    ## 3 A1        127 Batch1 A           Female Time1
    ## 4 A2        129 Batch1 A           Male   Time0
    ## 5 A2        130 Batch1 A           Male   Time1
    ## 6 A3          2 Batch2 B           Male   Time0

1.  **Define replicates within a batch**: two samples are replicate of each other if they belong to the same `Patient` and they were collected at the same `Time`.

``` r
patients_infos <- patients_infos %>%
  unite(Repl.Within, Patient, Time, sep = ".",remove=FALSE)
head(patients_infos)
```

    ## # A tibble: 6 x 7
    ##   Repl.Within Patient    ID Batch  DiseaseType Gender Time 
    ##   <chr>       <chr>   <dbl> <chr>  <chr>       <chr>  <chr>
    ## 1 A1.Time0    A1        126 Batch1 A           Female Time0
    ## 2 A1.Time2    A1        128 Batch1 A           Female Time2
    ## 3 A1.Time1    A1        127 Batch1 A           Female Time1
    ## 4 A2.Time0    A2        129 Batch1 A           Male   Time0
    ## 5 A2.Time1    A2        130 Batch1 A           Male   Time1
    ## 6 A3.Time0    A3          2 Batch2 B           Male   Time0

1.  Take unique combinations of the variables that I want to display in the structure of the data

``` r
reduced_infos <- patients_infos %>%
  group_by(Patient,DiseaseType,Time,Batch) %>% 
  summarise(Nsam=length(Repl.Within)) 
head(reduced_infos)
```

    ## # A tibble: 6 x 5
    ## # Groups:   Patient, DiseaseType, Time [6]
    ##   Patient DiseaseType Time  Batch   Nsam
    ##   <chr>   <chr>       <chr> <chr>  <int>
    ## 1 A1      A           Time0 Batch1     1
    ## 2 A1      A           Time1 Batch1     1
    ## 3 A1      A           Time2 Batch1     1
    ## 4 A10     A           Time1 Batch1     2
    ## 5 A10     A           Time2 Batch1     1
    ## 6 A11     A           Time0 Batch2     1

1.  Create variable used to spread the dataset from long to wide

``` r
reduced_infos <- reduced_infos %>%
  unite(Time.Batch, Time, Batch,sep=".")

infos_wide <- reduced_infos %>% spread(key = Time.Batch,value = Nsam)
length(unique(reduced_infos$Patient))
```

    ## [1] 38

1.  Set to NA if there are no samples available for a patient at one time point

``` r
infos_wide[is.na(infos_wide)] <- 0 
```

1.  The function `pheatmap()` wants a matrix as input to be displayed. The matrix needs rownames. At the moment `infos_wide` is a `tibble()` which does not allow roenames. I first `data.frame()` and when it is time to plot it I will only provide as input to `pheatmap()` numeric values.

``` r
infos_wide <- data.frame(infos_wide)
rownames(infos_wide) <- infos_wide$Patient
```

1.  Set snnotation for columns (time points) and rows (patients)

``` r
ann_columns <- data.frame(Batch=c("Batch1","Batch2",
                                    "Batch1","Batch2",
                                    "Batch1","Batch2"),
                           Time=c("Time0","Time0",
                                  "Time1","Time1",
                                  "Time2","Time2"))

ann_colors_wide <- list(Batch=c(Batch1 = "#762a83", Batch2 = "#1b7837"),
                        Time=c(Time0="#662506",Time1="#993404",Time2="#ec7014"),
                        DiseaseType = c(A="#78c679",B="#f7fcb9"))

rownames(ann_columns) <- colnames(infos_wide)[-c(1:2)]

# Row annotation
infos_wide$DiseaseType <- factor(infos_wide$DiseaseType,levels=c("A","B"),ordered = TRUE)
infos_wide <- infos_wide[order(infos_wide$DiseaseType),]
infos_wide$order <- 1:nrow(infos_wide)
infos_wide$order <- factor(infos_wide$order ,levels=infos_wide$order ,labels=infos_wide$Patient)

annotation_row <- data.frame(DiseaseType = infos_wide$DiseaseType)
rownames(annotation_row) <- infos_wide$order
```

These are the final elements that you need to plot the structure:

-   The matrix indicating how many replicates are at each time point for each patient

``` r
head(infos_wide[order(infos_wide$DiseaseType),-c(1,2,ncol(infos_wide))])
```

    ##     Time0.Batch1 Time0.Batch2 Time1.Batch1 Time1.Batch2 Time2.Batch1
    ## A1             1            0            1            0            1
    ## A10            0            0            2            0            1
    ## A11            0            1            0            0            0
    ## A12            0            0            0            2            0
    ## A13            1            0            2            0            4
    ## A15            0            1            0            2            0
    ##     Time2.Batch2
    ## A1             0
    ## A10            0
    ## A11            0
    ## A12            0
    ## A13            1
    ## A15            0

-   The annotation for the rows

``` r
head(annotation_row)
```

    ##     DiseaseType
    ## A1            A
    ## A10           A
    ## A11           A
    ## A12           A
    ## A13           A
    ## A15           A

-   The annotation for the columns

``` r
head(annotation_row)
```

    ##     DiseaseType
    ## A1            A
    ## A10           A
    ## A11           A
    ## A12           A
    ## A13           A
    ## A15           A

-   The colours for both annotation

``` r
ann_colors_wide
```

    ## $Batch
    ##    Batch1    Batch2 
    ## "#762a83" "#1b7837" 
    ## 
    ## $Time
    ##     Time0     Time1     Time2 
    ## "#662506" "#993404" "#ec7014" 
    ## 
    ## $DiseaseType
    ##         A         B 
    ## "#78c679" "#f7fcb9"

See the final result!
---------------------

``` r
pheatmap(infos_wide[order(infos_wide$DiseaseType),-c(1,2,ncol(infos_wide))],
         main = "Data Structure",
         cluster_cols=FALSE,cluster_rows=FALSE,
         annotation_col = ann_columns,
         annotation_row = annotation_row,
         annotation_colors = ann_colors_wide,
        breaks=c(0,0.9,1.9,2.9,3.9,4.9),col=c("#f1eef6","#74a9cf","#0570b0","blue","dark blue","black","grey"),show_colnames = FALSE,
        width = 11,height = 10)
```

<img src="How_R_You_R-LadiesMelbourne_code_and_tips_files/figure-markdown_github/unnamed-chunk-13-1.png" style="display: block; margin: auto;" />

Useful links to know more about heatmaps in R
---------------------------------------------

-   <http://www.sthda.com/english/articles/28-hierarchical-clustering-essentials/93-heatmap-static-and-interactive-absolute-guide/>
-   <http://www.bioconductor.org/packages/release/bioc/vignettes/ComplexHeatmap/inst/doc/s4.heatmap_annotation.html>

Violin plots with overlayed boxplots and coloured by group
==========================================================

**Author**: [Anna Quaglieri](https://github.com/annaquaglieri16)

Whenever I have to display to compare continuous variables this has become my favourite way to go! *Violin plot* + *boxplot* allows me to see both the quantiles and the overall density distribution that if can often be missed with only boxplots.

``` r
library(ggplot2)

data <- data.frame(Gene = rep(c("Gene1","Gene2","Gene3","Gene4"),each=46),
                   Counts = log2(rbinom(n = 46*4,size = 1000,prob = 0.3)),
                   CBF = sample(x = c("A","B"),size = 46*4,replace=TRUE))

dodge <- position_dodge(width = 1)
ggplot(data,aes(x=Gene,y=Counts,fill=CBF)) + theme_bw()  + theme(axis.text.x = element_text(angle = 0)) + geom_violin(trim=FALSE,position = dodge) + geom_boxplot(width=.1,position = dodge,show.legend = FALSE) +  labs(y="log2Counts") + facet_wrap(~Gene,scales="free_x")
```

![](How_R_You_R-LadiesMelbourne_code_and_tips_files/figure-markdown_github/unnamed-chunk-14-1.png)

Matching strings
================

**Author**: [Saskia Freytag](https://github.com/SaskiaFreytag)

Find Saskia's fabulous slides about all the ways to match strings in R [here](https://matchingstrings.netlify.com/#1).
