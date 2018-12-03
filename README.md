
`geohash_tools`
===============

This package provides tools for working with [Gustavo](https://github.com/niemeyer) [Niemeyer](https://twitter.com/gniemeyer)'s [geohash](https://en.wikipedia.org/wiki/Geohash) system of nestable, compact global coordinates based on [Z-order curves](https://en.wikipedia.org/wiki/Z-order_curve). The system consists of carving the earth into equally-sized rectangles (when projected into latitude/longitude space) and nesting this process recursively.

The C++ source code is largely copied from [Hiroaki Kawai](https://github.com/hkwi)'s excellent implementation of geohash encoding/decoding (built with a [Python API](https://github.com/hkwi/python-geohash)), with minor top-level tweaks to match the [`Rcpp`](http://www.rcpp.org/) API to R.

Encoding geohashes
------------------

Encoding is the process of turning latitude/longitude coordinates into geohash strings. For example, Parque Nacional Tayrona in Colombia is located at roughly 11.3113917 degrees of latitude, -74.0779006 degrees of longitude. This can be expressed more compactly as:

``` r
library(geohashTools)
gh_encode(11.3113917, -74.0779006)
```

    ## [1] "d65267"

These 6 characters identify this point on the globe to within 1.2 kilometers (east-west) and .6 kilometers (north-south).

The park is quite large, and this is too precise to cover the park; we can "zoom out" by reducing the precision (which is the number of characters in the output, `6` by default):

``` r
gh_encode(11.3113917, -74.0779006, precision = 5L)
```

    ## [1] "d6526"

### Public Art in Chicago

We can use this as a simple, regular level of spatial aggregation for spatial points data, e.g., counting presence of public art throughout the city of Chicago, as captured in [this dataset](https://data.cityofchicago.org/Parks-Recreation/Parks-Public-Art/sj6t-9cju) provided by the City:

``` r
tmp = tempfile(fileext = 'csv')
URL = paste0('https://data.cityofchicago.org/api/views/',
             'sj6t-9cju/rows.csv?accessType=DOWNLOAD')
download.file(URL, tmp)

library(data.table)
art = fread(tmp)

# count art by geohash
art[ , .N, by = .(geohash = gh_encode(LATITUDE, LONGITUDE, 5L))
     # only show the top 10
     ][order(-N)][1:10]
```

    ##     geohash  N
    ##  1:   dp3wm 46
    ##  2:   dp3wn 42
    ##  3:   dp3wt 16
    ##  4:   dp3wq 13
    ##  5:   dp3wk 10
    ##  6:   dp3wj  9
    ##  7:   dp3ty  9
    ##  8:   dp3w7  8
    ##  9:   dp3tw  6
    ## 10:   dp3tv  4

This is pretty impractical *per se* (where is `dp3wm`?); we'll return to this once we've introduced more functionality.

Decoding geohashes
------------------

The reverse of encoding geohashes is of course decoding them -- taking a given geohash string and converting it into global coordinates. For example, the Ethiopian coffee growing region of Yirgacheffe is roughly at `sc54v`:

``` r
gh_decode('sc54v')
```

    ## $latitude
    ## [1] 6.130371
    ## 
    ## $longitude
    ## [1] 38.21045

It can also be helpful to know just how precisely we've identified these coordinates; the `include_delta` argument gives the cell half-widths in both directions in addition to the cell centroid:

``` r
gh_decode('sc54v', include_delta = TRUE)
```

    ## $latitude
    ## [1] 6.130371
    ## 
    ## $longitude
    ## [1] 38.21045
    ## 
    ## $delta_latitude
    ## [1] 0.02197266
    ## 
    ## $delta_longitude
    ## [1] 0.02197266

In terms of latitude and longitude, all geohashes with the same precision have the same dimensions (though the physical size of the "rectangle" changes depending on the latitude); as such it's easy to figure out the cell half-widths from the precision alone using `gh_delta`:

``` r
gh_delta(5L)
```

    ## [1] 0.02197266 0.02197266

Geohash neighborhoods
---------------------

One unfortunate consequence of the geohash system is that, while geohashes that are lexicographically similar (e.g. `wxyz01` and `wxyz12`) are certainly close to one another, the converse is not true -- for example, `7gxyru` and `k58n2h` are neighbors! Put another way, small movements on the globe occasionally have visually huge jumps in the geohash-encoded output.

Fret not -- one tool for helping overcome this is the `gh_neighbors` function (`gh_neighbours` is also registered, for the Commonwealthy among us), which will return all of the geohashes adjacent to a given geohash (or vector of geohashes) at the same level of precision.

For example, Aung San Suu Kyi's childhood home is roughly at `w4urs5pc`, but this level of precision zooms in a bit too far. The geohash neighborhood thereof can be found with:

``` r
gh_neighbors('w4urs5pc')
```

    ## $southwest
    ## [1] "w4urs5p8"
    ## 
    ## $south
    ## [1] "w4urs5pb"
    ## 
    ## $southeast
    ## [1] "w4urs700"
    ## 
    ## $east
    ## [1] "w4urs701"
    ## 
    ## $northeast
    ## [1] "w4urs704"
    ## 
    ## $north
    ## [1] "w4urs5pf"
    ## 
    ## $northwest
    ## [1] "w4urs5pd"
    ## 
    ## $west
    ## [1] "w4urs5p9"

API to other GIS tools in R
---------------------------

-   `sp`
-   `sf`

See also
--------

You might also get benefit out of these online tools for working with geohashes:

-   <http://www.movable-type.co.uk/scripts/geohash.html>
-   <http://geohash.gofreerange.com/>
