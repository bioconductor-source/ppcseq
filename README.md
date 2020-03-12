Posterior predictive check for bulk RNA sequencing data
================

The input data set is a tidy representation of a differential gene
transcript abundance analysis

To install:

Before install, for linux systems, in order to exploit multi-threading,
from R write:

``` r
fileConn<-file("~/.R/Makevars")
writeLines(c( "CXX14FLAGS += -O3","CXX14FLAGS += -DSTAN_THREADS", "CXX14FLAGS += -pthread"), fileConn)
close(fileConn)
```

Then, install with

``` r
devtools::install_github("stemangiola/ppcseq")
```

You can get the test dataset with

``` r
ppcseq::counts 
```

    ## # A tibble: 394,821 x 9
    ##    sample  symbol   logCPM    LR   PValue        FDR value       W Label      
    ##    <chr>   <chr>     <dbl> <dbl>    <dbl>      <dbl> <int>   <dbl> <chr>      
    ##  1 10922PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   160 -0.129  High       
    ##  2 10935PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   150 -0.127  High       
    ##  3 10973PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   146 -0.426  High       
    ##  4 10976PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   347 -0.0164 High       
    ##  5 10985PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   175 -0.135  High       
    ##  6 11026PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   244  0.125  High       
    ##  7 11045PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   399 -0.0892 High       
    ##  8 11082PP SLC16A12   1.39  41.1 1.46e-10 0.00000274   100  0.261  Neoadjuvant
    ##  9 11086PP SLC16A12   1.39  41.1 1.46e-10 0.00000274    37 -0.132  Neoadjuvant
    ## 10 11103PP SLC16A12   1.39  41.1 1.46e-10 0.00000274    73  0.146  Neoadjuvant
    ## # … with 394,811 more rows

You can identify anrtefactual calls from your differential transcribt
anundance analysis, due to outliers.

``` r
counts.ppc = 
    ppcseq::counts %>%
    mutate(is_significant = FDR < 0.01) %>%
    identify_outliers(
        formula = ~ Label,
        .sample = sample, 
        .transcript = symbol,
        .abundance = value,
        .significance = PValue,
        .do_check = is_significant,
        percent_false_positive_genes = "5%"
    )
```

The new posterior predictive check has been added to the original data
frame

``` r
counts.ppc 
```

    ## # A tibble: 15 x 5
    ##    symbol   `sample wise data` plot   `ppc samples fail… `tot deleterious outli…
    ##    <chr>       <list<df[,14]>> <list>              <int>                   <int>
    ##  1 SLC16A12          [21 × 14] <gg>                    0                       0
    ##  2 CYP1A1            [21 × 14] <gg>                    1                       1
    ##  3 ART3              [21 × 14] <gg>                    1                       1
    ##  4 DIO2              [21 × 14] <gg>                    0                       0
    ##  5 OR51E2            [21 × 14] <gg>                    0                       0
    ##  6 MUC16             [21 × 14] <gg>                    0                       0
    ##  7 CCNA1             [21 × 14] <gg>                    0                       0
    ##  8 LYZ               [21 × 14] <gg>                    1                       1
    ##  9 PPM1H             [21 × 14] <gg>                    0                       0
    ## 10 SUSD5             [21 × 14] <gg>                    0                       0
    ## 11 TPRG1             [21 × 14] <gg>                    0                       0
    ## 12 EPB42             [21 × 14] <gg>                    0                       0
    ## 13 LRRC38            [21 × 14] <gg>                    0                       0
    ## 14 SUSD4             [21 × 14] <gg>                    0                       0
    ## 15 MMP8              [21 × 14] <gg>                    1                       1

The new data frame contains plots for each gene

We can visualise the top five differentially transcribed genes

``` r
counts.ppc %>% 
    slice(1:2) %>% 
    pull(plot) %>% 
    cowplot::plot_grid(plotlist = ., align = "v", ncol = 1, axis="b", rel_widths = 1 )
```

![](README_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->
