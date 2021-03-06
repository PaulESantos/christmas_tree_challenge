Christmas Tree Challenge
================
Roman M. Link

## Background

This project is a special challenge for the Christmas edition of the
monthly statistics seminar I organize together with
[Hego-CCTB](https://github.com/Hego-CCTB), a fellow PhD student at the
Institute of Botany of the University of Würzburg. Anyone who’s
interested in improving his or her statistical prowess and R skills is
welcome, so if you are located in Würzburg and want to give it a shot,
have a look at the [course
repository](https://github.com/r-link/Julius_von_Stats) for upcoming
dates (but be forewarned that the repository is a work in progress, and
our seminar is catering to the needs of the PhD students at our
institute, i.e. mostly on beginner level).

After reading an [interesting article about plant
architecture](http://www.indefenseofplants.com/blog/2017/7/2/plant-architecture-and-its-evolutionary-implications)
on the great [In Defense of Plants](http://www.indefenseofplants.com/)
blog, I knew that one day I’d *have* to play around with models of tree
branching patterns. Tree architectural models are a wonderful example
how complex architectural features can arise from surprisingly simple
rules. At the same time, they are fun to play with, and as a by-product,
their predictions are more often than not quite aesthetically pleasing.
Quite a nice topic for our Christmas session, don’t you think?

## The Challenge

For the challenge, I prepared a little script (`grow_tree.R` in the `R`
folder) that generates two-dimensional branching patterns based on
simple user-specified rules. In each iteration of the model, each
terminal segment undergoes a dichotomous branching. Each new segment
gets an unique ID that allows to trace all parents. In addition, all new
segments get a user-specified survival probability, length and angle of
inclination that all can be expressed as a function of all other
segment-specific information.

The tasks is as follows:

| The Christmas Tree Challenge: Rules                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1\.   Use `grow_tree()` with a set of functions for survival probability, segment length and angle that results in *branching* <br>       *patterns typical for conifers* (i.e. to ‘grow a Christmas tree’), <br> 2.   Use all available ggplot2 magic to make the tree look as realistical as possible, <br> 3.   Add as much graphical sugar as possible to decorate it in order to plot the *perfect Christmas tree*. <br> <br> Biological correctness does not matter for the challenge – everything is allowed, provided that the plots are **based only on data generated with the `grow_tree()` function** in the script in this repository. |

The most realistical and beautiful trees win (though we are neither sure
about the price nor how to determine the winner).

## How to use `grow_tree.R`

### Preparations

The script is in the `R` folder of this repository. To use it, just make
sure that the `tidyverse` and `rlang` packages are loaded, and load the
script via `source()`.

``` r
# load packages
library(tidyverse)
library(rlang)

# source grow_tree.R script (assuming it is in the /R subdirectory of the working directory)
source("R/grow_tree.R")
```

The script then should take care of all things. You just have to provide
it with its desired inputs, i.e. functions for survival probability,
segment length and angle.

### Simple example

As a motivating example, here’s how a call to `grow_tree()` might look
like (if you want to run it yourself without copying and pasting, all
examples are in the script `01_christmas_tree_example.R` in the root
folder of the repository):

``` r
# define simple angle, length and survival functions
angle_fun1  <- function(angle) angle + c(-0.1 * pi, 0.1 * pi) 
               # angle: parent angle +- 0.1pi 
length_fun1 <- function(length) 0.85 * length
               # length: 85% of parent length
surv_fun1   <- function() 0.9
               # survival probability: 90% for both children

# define random seed (for reproducibility)
set.seed(12345)

# grow your first tree (using 15 iterations)
tree1 <- grow_tree(n_iter     = 10, 
                   angle_fun  = angle_fun1(angle), 
                   length_fun = length_fun1(length), 
                   surv_fun   = surv_fun1(),
                   verbose    = FALSE) # do not print progress

# inspect the output
tree1
```

    ## # A tibble: 507 x 13
    ##    generation id    length  angle parent_id parent_length parent_angle
    ##         <dbl> <chr>  <dbl>  <dbl> <chr>             <dbl>        <dbl>
    ##  1          1 1      1      0     <NA>             NA           NA    
    ##  2          2 11     0.85  -0.314 1                 1            0    
    ##  3          2 12     0.85   0.314 1                 1            0    
    ##  4          3 111    0.722 -0.628 11                0.85        -0.314
    ##  5          3 121    0.722  0     12                0.85         0.314
    ##  6          3 112    0.722  0     11                0.85        -0.314
    ##  7          3 122    0.722  0.628 12                0.85         0.314
    ##  8          4 1111   0.614 -0.942 111               0.722       -0.628
    ##  9          4 1211   0.614 -0.314 121               0.722        0    
    ## 10          4 1121   0.614 -0.314 112               0.722        0    
    ## # … with 497 more rows, and 6 more variables: x0 <dbl>, y0 <dbl>,
    ## #   x1 <dbl>, y1 <dbl>, surv_prob <dbl>, surv <lgl>

Details for the specification of `angle_fun`, `length_fun` and
`surv_fun` are described below.

If you want to have an idea what the simulated tree looks like,
`plot_tree` offers a simple automated way of visually inspecting the
plotted tree:

``` r
# plot with the plot_tree function
# (ony_living = TRUE excludes all dead segments from the plot)
plot_tree(tree1, only_living = TRUE)
```

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

As you see, even with only 10 iterations and very simplistic branching
rules, the output looks very similar to a real plant. It has a vibe of
very early land plants, though, and sort of reminds me of the branching
patterns of plants like
[Aglaophyton](https://en.wikipedia.org/wiki/Aglaophyton).

### Structure of ‘treetab’ objects

The output of `grow_tree()` is an object of class `treetab`, which is
basically just a tibble with an appended class. Its columns are as
follows:

| Column         | Type                   | Description                                                                                                                                                                                                                                                                                                                                 |
| -------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| generation     | double                 | which generation of growth is a segment from? (ranges from 1 to n\_iter)                                                                                                                                                                                                                                                                    |
| id             | character              | unique identifier of each segment. Contains the information about all parents segments in the dichotomous branching segment. In each iteration, a “1” for the first and a “2” for the second segment is appended to the parent semgent’s ID. Depending of how your tree is built, this variable can be useful to determine branching orders |
| length         | double                 | segment length (in arbitrary units)                                                                                                                                                                                                                                                                                                         |
| angle          | double                 | angle to the y axis (counter-clockwise, in radians)                                                                                                                                                                                                                                                                                         |
| parent\_id     | character              | ID of the parent segment                                                                                                                                                                                                                                                                                                                    |
| parent\_length | double                 | length of the parent segment                                                                                                                                                                                                                                                                                                                |
| parent\_angle  | double                 | angle of the parent segment                                                                                                                                                                                                                                                                                                                 |
| x0             | double                 | x postition of the basipetal (“lower”) end of the segment (equal to the coordinates of the terminal end of parent segment).                                                                                                                                                                                                                 |
| y0             | double                 | y postition of the basipetal (“lower”) end of the segment (equal to the coordinates of the terminal end of parent segment).                                                                                                                                                                                                                 |
| x1             | double                 | x postition of the acropetal (“upper”) end of the segment (computed from length, angle, x0 and y0).                                                                                                                                                                                                                                         |
| y1             | double                 | y postition of the acropetal (“upper”) end of the segment (computed from length, angle, x0 and y0).                                                                                                                                                                                                                                         |
| surv\_prob     | double between 0 and 1 | survival probability of a segment (evaluated once for each new generation of segments)                                                                                                                                                                                                                                                      |
| surv           | logical                | outcome of a random draw with probability surv\_prob - only segments with surv == TRUE will grow in the next generation.                                                                                                                                                                                                                    |

### Defining update functions

`angle_fun`, `length_fun` and `surv_fun` can be calls to arbitrary
functions returning segment angles, lengths and survival probabilities,
respectively. They are evaluated in the environment of the new-formed
segment, for each parent segment separately. Any functions are valid, as
long as they return a numeric value of length 1 (i.e. same value for
both children) or 2 (separate values per child). The outcome of
`surv_fun` is additionally bound to the range from 0 to 1, as it
represents a probability. If you want to keep one or several values
fixed, you can also directly specify numeric values, e.g. `surv_fun
= 1`.

As the variables are evaluated in the environment of the parent segment,
the functions can take any of the variables listed in the previous
section as arguments (the input is to be treated as vectors of length 2,
i.e. one value per child).

The functions are evaluated in the order angle - length - survival, so
length can be set in dependence of the new segment’s angle of
inclination, and survival can be updated using both the current
segment’s length and angle (which can be very useful if you e.g. want
to treat side branches differently). Before the variables are updated,
they have the value of the parent segment, so when updating `length` and
`angle` their names can be used as shorthands for `parent_length` and
`parent_angle`.

| Function     | Input                  | Output                                 | Order of evaluation | Details                                                       |
| ------------ | ---------------------- | -------------------------------------- | :-----------------: | ------------------------------------------------------------- |
| `angle_fun`  | any vars. in `treetab` | numeric of length 1 or 2               |          1          | angle to the y-axis in radians in counter-clockwise direction |
| `length_fun` | any vars. in `treetab` | numeric of length 1 or 2               |          2          | segment length                                                |
| `surv_fun`   | any vars. in `treetab` | numeric between 0 and 1, length 1 or 2 |          3          | survival probability of each newly created segment            |

As the function calls are quasi-quotated and evaluated in the
environment of the parent node, the specification of updating rules can
be done by simple function calls (see examples).

## A more elaborate example

Here’s an example resulting in a bit more realistic tree. Again, keep in
mind that for the challenge, the model is not meant to be biologically
realistic - it is just about having fun, and producing beautiful trees…

``` r
# angle defined as a function of generation and parent angle;
# symmetric bifurcation with increasing average angle and variability in
# angles higher up in the canopy
angle_fun2  <- function(angle, generation){
  # weights that get bigger which each generation
  w  <- 1 - 1 / generation
  # get absolute deviations from parent segment angle
  a0 <- rnorm(1, 0.12 * w * pi, 0.05 * w * pi)
  # return new angles with randomized order
  angle + sample(c(-a0, a0))
  }

# length as a function of parent length, decreasing on average by 10 percent
# with stochastic variation
length_fun2 <- function(length) rnorm(2, 0.9, 0.05) * length
# survival function as a function of generation and number of iterations -
# one of the branches survives with a probability of 95%, the 'survival' 
# probability of the other branch increases with tree age (looks more
# realistic because it seems like broken off branches)
surv_fun2   <- function(generation, n_iter) c(.75 - (n_iter - generation[1]) / (2 * n_iter), .95)

# set random seed 
set.seed(999)

# grow tree for 25 iterations
tree2 <- grow_tree(n_iter = 25, 
                   angle_fun2(angle, generation), 
                   length_fun2(length), 
                   surv_fun2(generation, 25), # note that n_iter has to be 
                                              # specified explicitly because it
                                              # is not in the treetab
                   verbose = FALSE)

# plot tree
plot_tree(tree2)
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

…if this doesn’t look like an angiosperm tree to you, I am not sure what
does.

## Modifying plots of trees

While `plot_tree()` is useful to have a first glance on a tree object,
if you have found your desired Christmas tree anatomy and want to start
glamming it up, you will find that you need more flexibility.
Internally, `plot_tree()` just gets the tree object in a form that is
digestible for `ggplot()`, and then plots it with some modifications to
scales and themes.

If you want to set the plotting parameters more flexibly, you can use
the `clean_tree()` function yourself to convert `treedat` objects into a
long table format that is more appropriate for plotting:

``` r
# clean the second tree
tree2_clean <- clean_tree(tree2)

# inspect the clean tree
tree2_clean
```

    ## # A tibble: 130,770 x 12
    ##    generation id    length   angle parent_id parent_length parent_angle
    ##  *      <dbl> <chr>  <dbl>   <dbl> <chr>             <dbl>        <dbl>
    ##  1          1 1      1      0      <NA>             NA          NA     
    ##  2          2 11     0.872  0.210  1                 1           0     
    ##  3          2 12     0.806 -0.210  1                 1           0     
    ##  4          3 112    0.792  0.360  11                0.872       0.210 
    ##  5          3 122    0.671  0.180  12                0.806      -0.210 
    ##  6          4 1121   0.699 -0.0292 112               0.792       0.360 
    ##  7          4 1221   0.566  0.0424 122               0.671       0.180 
    ##  8          4 1122   0.724  0.749  112               0.792       0.360 
    ##  9          4 1222   0.625  0.318  122               0.671       0.180 
    ## 10          5 12211  0.495 -0.282  1221              0.566       0.0424
    ## # … with 130,760 more rows, and 5 more variables: surv_prob <dbl>,
    ## #   surv <lgl>, pos <dbl>, x <dbl>, y <dbl>

Now there is only one column for x and y, respectively, while a new
column (pos) indicates whether a row refers to the basal (0) or apical
end (1) of a segment. `plot_tree()` takes this output and plots it with
the following settings:

``` r
# create plot of clean tree manually
p1 <- tree2_clean %>% 
    ggplot(aes(x = x, 
               y = y, 
               group = id, 
               size = 1 / generation,
               color = generation)) +
    geom_line(lineend = "round") +
    scale_color_tree() +
    theme_void() +
    theme(legend.position = "none") + 
    coord_equal()
```

Grouping by ID is necessary to avoid ending up with a huge spaghetti
plot. Conditioning size on generation makes sure that the terminal
branches are narrower than the stem. The color scheme I defined
(`scale_color_tree()`) is just a collection of shades of brown and green
that gets greener with higher values, In general, it seems to work
pretty well when coloring by generation. `theme_void()` and
`theme(legend.position = "none")` remove all non-data elements from the
plot, and `coord_equal()` makes sure the horizontal and vertical
dimensions of the tree are maintained to avoid distorted plots for very
wide or narrow trees.

Of course, you can modify the plot in any way you want. As mentioned
above, the only condition for the challenge is that all the data used in
the plot have to be taken from the tree object.

You can e.g. beautify the tree by adding some baubles at a number of
random terminal branch ends:

``` r
# set random seed
set.seed(13579)
# extract 30 random basal ends of the terminal branches
baubles <- filter(tree2_clean, 
                  generation == 25,
                  pos == 0) %>% 
  sample_n(30)

# add baubles to the tree
p1 +
  # red baubles with black outlines
  geom_point(data = baubles, aes(x = x, y = y), 
             fill = "#AA1243", col = 1, pch = 21, size = 5) +
  # white transparent shine (with a small offset to avoid the center)
  geom_point(data = baubles, aes(x = x, y = y), 
             col = "white", alpha = 0.3, size = 1,
             position = position_nudge(x = -0.08, y = 0.08)
            )
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Obviously, this is not the Christmas tree we were looking for. It might
pass as an apple tree, though. Anyway, we are moving in the right
direction. Further improvements can e.g. achieved by filtering the
dataset by generation and adding differently colored layers, possibly
using different geoms…

## Where to go?

Here’s an animation of a tree created with a different set of update
functions: ![A nice tree](animation/animation.gif)

While the outcome mimics the architecture of a ‘typical Christmas tree’
relatively well, there are better solutions for sure, and you could
spend much more time decorating it…

Feel free to play around with the script to reproduce other plant growth
patterns (palms, anyone?). Or maybe even use it as a blueprint for some
more serious models of plant architecture?

By the way, it should go without saying that everything about this
challenge is just about having fun playing with code, so please do not
get agitated about the religious connotations. As for me, I’d be
absolutely fine if you prefer to make a Hannukah bush or an evil satanic
black metal tree (kudos if you do the latter)…

So long, good luck and goodspeed, and may you trees turn out nice and
healthy\!
