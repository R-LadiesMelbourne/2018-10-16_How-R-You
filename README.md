
-   [Packages to install](#packages-to-install)
-   [Reference material for the following tutorial](#reference-material-for-the-following-tutorial)
-   [Downloading tweets with `twitteR`](#downloading-tweets-with-twitter)

Packages to install
===================

``` r
install.packages("twitteR")
install.packages("wordcloud2")
install.packages("tidyverse")
install.packages("tidytext")
install.packages("knitr")
install.packages("plotly")
devtools::install_github("ropenscilabs/icon") # to insert icons
devtools::install_github("hadley/emo") # to insert emoji
```

``` r
library(knitr)
library(magick)
```

    ## Linking to ImageMagick 6.9.9.39
    ## Enabled features: cairo, fontconfig, freetype, lcms, pango, rsvg, webp
    ## Disabled features: fftw, ghostscript, x11

``` r
library(png)
library(grid)
library(emo)
library(icon)
library(twitteR)
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
    ## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ───────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter()   masks stats::filter()
    ## ✖ dplyr::id()       masks twitteR::id()
    ## ✖ dplyr::lag()      masks stats::lag()
    ## ✖ dplyr::location() masks twitteR::location()

Reference material for the following tutorial
=============================================

-   [Setting up the Twitter R package for text analytics](https://www.r-bloggers.com/setting-up-the-twitter-r-package-for-text-analytics/)
-   [Obtaining and using access tokens](https://cran.r-project.org/web/packages/rtweet/vignettes/auth.html)
-   [Text mining with R](https://www.tidytextmining.com/index.html)

-   **Refer to `Social_Media_Setup_APIs.Rmd` to setup your Twitter API and connect your R session with Twitter**

Downloading tweets with `twitteR`
=================================

Once you have connected to twitter <!--html_preserve--><i class="fab  fa-twitter "></i><!--/html_preserve--> and accessed your API, you are ready to download tweets. Let's try to return the 3 most recent tweets that contain the \#useR2018 hashtag.
