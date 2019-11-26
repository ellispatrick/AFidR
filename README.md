
<!-- README.md is generated from README.Rmd. Please edit that file -->

# AFid

Autofluorescence is a long-standing problem that has hindered
fluorescence microscopy image analysis. To address this, we have
developed a method that identifies and can exclude autofluorescent signals
from multi-channel images post acquisition.

## Installation

``` r
library(devtools)
devtools::install_github("ellispatrick/AFid")
```

## A quick example

``` r
library(AFid)
#> Loading required package: moments
#> Loading required package: EBImage
set.seed(51773)
## Read in images.
imageFile1 = system.file("extdata","ImageB.CD3.tif", package = "AFid")
imageFile2 = system.file("extdata","ImageB.CD11c.tif", package = "AFid")
im1 <- EBImage::readImage(imageFile1)
im2 <- EBImage::readImage(imageFile2)

## Rescale the images.
im1 = im1/max(im1)
im2 = im2/max(im2)

combined <- EBImage::rgbImage(green=sqrt(im1), red=sqrt(im2))
EBImage::display(combined, all = TRUE, method = 'raster')
```

![](man/figures/README-unnamed-chunk-2-1.png)<!-- -->

``` r


## Create masks using EBImage.

# Find tissue area
tissue1 = im1 > 2*min(im1)
tissue2 = im2 > 2*min(im2)

# Calculate thresholds
imThreshold1 <- mean(im1[tissue1]) + 2*sd(im1[tissue1])
imThreshold2 <- mean(im2[tissue2]) + 2*sd(im2[tissue2])

# Calculate masks.
mask1 <- EBImage::bwlabel(im1 > imThreshold1)
mask2 <- EBImage::bwlabel(im2 > imThreshold2)

## Calculate intersection mask
mask <- intMask(mask1,mask2)


## Calculate textural features.
df <- afMeasure(im1, im2, mask)
#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

## Alternatively
## Correlation only
# afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, corr = 0.6)

## Clustering with given k
# afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, k = 6)

## Clustering with estimated k.
afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, k = 20, kAuto = TRUE)


## Exclude autofluorescence from images (Use ImageJ for halo removal)
im1AFExcluded <- im1
im2AFExcluded <- im2
im1AFExcluded[afMask != 0] <- 0
im2AFExcluded[afMask != 0] <- 0

combinedExcluded <- EBImage::rgbImage(green = sqrt(im1AFExcluded), red = sqrt(im2AFExcluded))
img_comb = EBImage::combine(combined, combinedExcluded)
EBImage::display(img_comb, all = TRUE, method = 'raster',nx = 2)
```

![](man/figures/README-unnamed-chunk-2-2.png)<!-- -->

``` r


## Or
## Exclude AF ROIs

exclude1 = unique(mask1[afMask>0])
mask1Excluded = mask1
mask1Excluded[mask1Excluded%in%exclude1] = 0
exclude2 = unique(mask2[afMask>0])
mask2Excluded = mask2
mask2Excluded[mask2Excluded%in%exclude2] = 0


## OR
## A more agressive strategy to exclude autofluorescence from images as an alternative to the halo removal implemented in ImageJ
im1AFExcludedv2 <- im1
im2AFExcludedv2 <- im2
im1AFExcludedv2[!mask1%in%mask1Excluded|!mask2%in%mask2Excluded] <- 0
im2AFExcludedv2[!mask1%in%mask1Excluded|!mask2%in%mask2Excluded] <- 0

combinedExcludedv2 <- EBImage::rgbImage(green = sqrt(im1AFExcludedv2), red = sqrt(im2AFExcludedv2))
img_combv2 = EBImage::combine(combined, combinedExcludedv2)
EBImage::display(img_combv2, all = TRUE, method = 'raster',nx = 2)
```

![](man/figures/README-unnamed-chunk-2-3.png)<!-- -->
