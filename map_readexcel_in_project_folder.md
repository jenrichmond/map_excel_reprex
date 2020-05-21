load packages
=============

    library(readxl)
    library(tidyverse)
    library(here)

I have mutliple excel files that live in a data folder. I want to read
them all into R, adding a filename variable to each. I found the
original solution to the problem at this [stackflow
link](https://stackoverflow.com/questions/47540471/load-multiple-excel-files-and-name-object-after-a-file-name)

Original solution
=================

    # define a path where the data lives

    file_path <- here("data")

    # get a list of files at that file_path
    file_list <- list.files(path = file_path, pattern='*.xlsx')

    # use map to read_excel all of those files, adding a file_name variable to each one

    list_raw <- file_list %>% 
      map(~ read_excel(path = .x, range = "A1:C5")  %>%
            mutate("file_name" = .x))
    #> Error: `path` does not exist: 'fruitdata1.xlsx'

Trouble shooting the error
==========================

6 options to deal with paths by Jenny Bryan
-------------------------------------------

Here’s a troubleshooting chunk that reveals the working directory and
`file_list`. The path problem is going to be that `list.files()`, by
default, only returns file names. But you need a path relative to
whatever the working directory will be at render time or, even better, a
resilient path determined at render time by here.

    getwd()
    #> [1] "/Users/jenny/Desktop/map_excel_reprex"

    file_list
    #> [1] "fruitdata1.xlsx" "fruitdata2.xlsx" "fruitdata3.xlsx" "fruitdata4.xlsx"

    file.exists(file_list)
    #> [1] FALSE FALSE FALSE FALSE

#### Option 1

Be explicit about the path, using `path = here("data", .x)` inside
`read_excel()`

    file_list %>% 
      map(~ read_excel(path = here("data", .x), range = "A1:C5")  %>%
            mutate("file_name" = .x))
    #> [[1]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  3635 fruitdata1.xlsx
    #> 2 orange       orange  3567 fruitdata1.xlsx
    #> 3 grapes       green   3564 fruitdata1.xlsx
    #> 4 strawberries red     3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  7578 fruitdata2.xlsx
    #> 2 orange       orange  8546 fruitdata2.xlsx
    #> 3 grapes       green   6478 fruitdata2.xlsx
    #> 4 strawberries red     4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  4345 fruitdata3.xlsx
    #> 2 orange       orange  2456 fruitdata3.xlsx
    #> 3 grapes       green   3564 fruitdata3.xlsx
    #> 4 strawberries red     5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  5684 fruitdata4.xlsx
    #> 2 orange       orange  6754 fruitdata4.xlsx
    #> 3 grapes       green   3423 fruitdata4.xlsx
    #> 4 strawberries red     5674 fruitdata4.xlsx

#### Option 2

Use `here("data", .x)` on `file_list` on the way in. Now you’ll want to
use `basename()` on the paths before adding to the data frame.

> Q: What does
> [basename](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/basename)
> do? A: removes all of the path up to and including the last path
> separator (if any). Without it, the filename column ends up having the
> whole path listed
> (/Users/jenny/Desktop/map\_excel\_reprex/data/fruitdata1.xlsx)

    file_list %>% 
      here("data", .) %>% 
      map(~ read_excel(path = .x, range = "A1:C5")  %>%
            mutate("file_name" = basename(.x)))
    #> [[1]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  3635 fruitdata1.xlsx
    #> 2 orange       orange  3567 fruitdata1.xlsx
    #> 3 grapes       green   3564 fruitdata1.xlsx
    #> 4 strawberries red     3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  7578 fruitdata2.xlsx
    #> 2 orange       orange  8546 fruitdata2.xlsx
    #> 3 grapes       green   6478 fruitdata2.xlsx
    #> 4 strawberries red     4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  4345 fruitdata3.xlsx
    #> 2 orange       orange  2456 fruitdata3.xlsx
    #> 3 grapes       green   3564 fruitdata3.xlsx
    #> 4 strawberries red     5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  5684 fruitdata4.xlsx
    #> 2 orange       orange  6754 fruitdata4.xlsx
    #> 3 grapes       green   3423 fruitdata4.xlsx
    #> 4 strawberries red     5674 fruitdata4.xlsx

#### Option 3

Use `list.files(full.names = TRUE)` when capturing the file paths.
Again, use `basename()` on the paths before adding to the data frame.

Remember- here is how I made the first file\_list. It is just a list of
file names.

    file_list1 <- list.files(path = file_path, pattern='*.xlsx')

    file_list1
    #> [1] "fruitdata1.xlsx" "fruitdata2.xlsx" "fruitdata3.xlsx" "fruitdata4.xlsx"

Here is how JennyB suggests you make a file list. Specifying full.names
= TRUE gets the whole path.


    file_list2 <- list.files(path = here("data"), pattern='*.xlsx', full.names = TRUE)

    file_list2
    #> [1] "/Users/jenny/Desktop/map_excel_reprex/data/fruitdata1.xlsx"
    #> [2] "/Users/jenny/Desktop/map_excel_reprex/data/fruitdata2.xlsx"
    #> [3] "/Users/jenny/Desktop/map_excel_reprex/data/fruitdata3.xlsx"
    #> [4] "/Users/jenny/Desktop/map_excel_reprex/data/fruitdata4.xlsx"

Using the file\_list2 (with the whole path) and then basename (to strip
the bit of the path you dont need in the filename column).


    file_list2 %>% 
      map(~ read_excel(path = .x, range = "A1:C5")  %>%
            mutate("file_name" = basename(.x)))
    #> [[1]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  3635 fruitdata1.xlsx
    #> 2 orange       orange  3567 fruitdata1.xlsx
    #> 3 grapes       green   3564 fruitdata1.xlsx
    #> 4 strawberries red     3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  7578 fruitdata2.xlsx
    #> 2 orange       orange  8546 fruitdata2.xlsx
    #> 3 grapes       green   6478 fruitdata2.xlsx
    #> 4 strawberries red     4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  4345 fruitdata3.xlsx
    #> 2 orange       orange  2456 fruitdata3.xlsx
    #> 3 grapes       green   3564 fruitdata3.xlsx
    #> 4 strawberries red     5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  5684 fruitdata4.xlsx
    #> 2 orange       orange  6754 fruitdata4.xlsx
    #> 3 grapes       green   3423 fruitdata4.xlsx
    #> 4 strawberries red     5674 fruitdata4.xlsx

#### Option 4

The `fs` package has useful functions for dealing with path issues.

> Q: What does `fs` stand for? A: File systems; the `fs` package is a
> cross-platform interface to file system operations

Use `fs::dir_ls()` instead of `list.files()`, which IMO has better
defaults than `list.files()`. Now we’ll switch to `fs::path_file()` to
shorten the paths before adding to the data frame.

    library(fs)

    file_list_fs <- dir_ls(here("data"), glob = "*.xlsx")

    file_list_fs
    #> /Users/jenny/Desktop/map_excel_reprex/data/fruitdata1.xlsx
    #> /Users/jenny/Desktop/map_excel_reprex/data/fruitdata2.xlsx
    #> /Users/jenny/Desktop/map_excel_reprex/data/fruitdata3.xlsx
    #> /Users/jenny/Desktop/map_excel_reprex/data/fruitdata4.xlsx

> Q: what is “glob”? A: glob patterns specify sets of filenames with
> wildcard characters, see
> \[Wikipedia\](<a href="https://en.wikipedia.org/wiki/Glob_(programming)" class="uri">https://en.wikipedia.org/wiki/Glob_(programming)</a>


    file_list_fs %>% 
      map(~ read_excel(path = .x, range = "A1:C5")  %>%
            mutate("file_name" = fs::path_file(.x)))
    #> $`/Users/jenny/Desktop/map_excel_reprex/data/fruitdata1.xlsx`
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  3635 fruitdata1.xlsx
    #> 2 orange       orange  3567 fruitdata1.xlsx
    #> 3 grapes       green   3564 fruitdata1.xlsx
    #> 4 strawberries red     3367 fruitdata1.xlsx
    #> 
    #> $`/Users/jenny/Desktop/map_excel_reprex/data/fruitdata2.xlsx`
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  7578 fruitdata2.xlsx
    #> 2 orange       orange  8546 fruitdata2.xlsx
    #> 3 grapes       green   6478 fruitdata2.xlsx
    #> 4 strawberries red     4674 fruitdata2.xlsx
    #> 
    #> $`/Users/jenny/Desktop/map_excel_reprex/data/fruitdata3.xlsx`
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  4345 fruitdata3.xlsx
    #> 2 orange       orange  2456 fruitdata3.xlsx
    #> 3 grapes       green   3564 fruitdata3.xlsx
    #> 4 strawberries red     5647 fruitdata3.xlsx
    #> 
    #> $`/Users/jenny/Desktop/map_excel_reprex/data/fruitdata4.xlsx`
    #> # A tibble: 4 x 4
    #>   fruit        colour votes file_name      
    #>   <chr>        <chr>  <dbl> <chr>          
    #> 1 banana       yellow  5684 fruitdata4.xlsx
    #> 2 orange       orange  6754 fruitdata4.xlsx
    #> 3 grapes       green   3423 fruitdata4.xlsx
    #> 4 strawberries red     5674 fruitdata4.xlsx

#### Option 5

If `data` held *only* Excel files you could use `fs::dir_map()`. But it
won’t work as long as that `.Rmd` is in there.


    foo <- function(full_path) {
      read_excel(path = full_path, range = "A1:C5") %>%
        mutate("file_name" = path_file(full_path))
    }

    dir_map("data", foo)

#### Option 6

I suspect you will eventually want to row bind these data frames? In
that case, I think the most pleasing work flow is with
`purrr::map_dfr()`. If your paths are well-named from the start, its
`id` argument accomplishes what you doing with `mutate()`.

    file_list_fs <- dir_ls(here("data"), glob = "*.xlsx")
    names(file_list_fs) <- path_file(file_list_fs)

    file_list_fs %>% 
      map_dfr(~ read_excel(path = .x, range = "A1:C5"), .id = "file_name")
    #> # A tibble: 16 x 4
    #>    file_name       fruit        colour votes
    #>    <chr>           <chr>        <chr>  <dbl>
    #>  1 fruitdata1.xlsx banana       yellow  3635
    #>  2 fruitdata1.xlsx orange       orange  3567
    #>  3 fruitdata1.xlsx grapes       green   3564
    #>  4 fruitdata1.xlsx strawberries red     3367
    #>  5 fruitdata2.xlsx banana       yellow  7578
    #>  6 fruitdata2.xlsx orange       orange  8546
    #>  7 fruitdata2.xlsx grapes       green   6478
    #>  8 fruitdata2.xlsx strawberries red     4674
    #>  9 fruitdata3.xlsx banana       yellow  4345
    #> 10 fruitdata3.xlsx orange       orange  2456
    #> 11 fruitdata3.xlsx grapes       green   3564
    #> 12 fruitdata3.xlsx strawberries red     5647
    #> 13 fruitdata4.xlsx banana       yellow  5684
    #> 14 fruitdata4.xlsx orange       orange  6754
    #> 15 fruitdata4.xlsx grapes       green   3423
    #> 16 fruitdata4.xlsx strawberries red     5674
