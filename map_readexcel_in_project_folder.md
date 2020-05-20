load packages
=============

    library(readxl)
    library(tidyverse)
    #> ── Attaching packages ─────────────────────────────────── tidyverse 1.2.1.9000 ──
    #> ✓ ggplot2 3.2.1           ✓ purrr   0.3.4.9000 
    #> ✓ tibble  3.0.1           ✓ dplyr   0.8.99.9002
    #> ✓ tidyr   1.0.3.9000      ✓ stringr 1.4.0      
    #> ✓ readr   1.3.1           ✓ forcats 0.4.0
    #> Warning: package 'tibble' was built under R version 3.6.2
    #> ── Conflicts ─────────────────────────────────────────── tidyverse_conflicts() ──
    #> x dplyr::filter() masks stats::filter()
    #> x dplyr::lag()    masks stats::lag()
    library(here)
    #> here() starts at /Users/jenny/tmp/map_excel_reprex

Goal to read in multiple excel files, adding a filename variable to
each.

Solution from
<a href="https://stackoverflow.com/questions/47540471/load-multiple-excel-files-and-name-object-after-a-file-name" class="uri">https://stackoverflow.com/questions/47540471/load-multiple-excel-files-and-name-object-after-a-file-name</a>

set up path and files list
==========================

Use here::here to define path to data

    file_path <- here("data")

    file_list <- list.files(path = file_path, pattern='*.xlsx')

Here’s a troubleshooting chunk that reveals working directory and
`file_list`. The path problem is going to be that `list.files()`, by
default, only returns file names. But you need a path relative to
whatever the working directory will be at render time or, even better, a
resilient path determined at render time by here.

    getwd()
    #> [1] "/Users/jenny/tmp/map_excel_reprex"

    file_list
    #> [1] "fruitdata1.xlsx" "fruitdata2.xlsx" "fruitdata3.xlsx" "fruitdata4.xlsx"

    file.exists(file_list)
    #> [1] FALSE FALSE FALSE FALSE

#### map read\_excel

Take the list of files, read\_excel each file A1:D5, adding a filename
variable, return a list.

    list_raw <- file_list %>% 
      map(~ read_excel(path = .x, range = "A1:D5")  %>%
            mutate("file_name" = .x))
    #> Error: `path` does not exist: 'fruitdata1.xlsx'

    # test that your list items are what you want 
    fruitdata1 <- list_raw[[1]]
    #> Error in eval(expr, envir, enclos): object 'list_raw' not found

### Various fixes for the path problems

Use `path = here("data", .x)` inside `read_excel()`:

    file_list %>% 
      map(~ read_excel(path = here("data", .x), range = "A1:D5")  %>%
            mutate("file_name" = .x))
    #> [[1]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    240  3635 fruitdata1.xlsx
    #> 2 orange       orange    375  3567 fruitdata1.xlsx
    #> 3 grapes       green     290  3564 fruitdata1.xlsx
    #> 4 strawberries red       187  3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    274  7578 fruitdata2.xlsx
    #> 2 orange       orange    363  8546 fruitdata2.xlsx
    #> 3 grapes       green     678  6478 fruitdata2.xlsx
    #> 4 strawberries red       343  4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    896  4345 fruitdata3.xlsx
    #> 2 orange       orange    453  2456 fruitdata3.xlsx
    #> 3 grapes       green     765  3564 fruitdata3.xlsx
    #> 4 strawberries red       345  5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    545  5684 fruitdata4.xlsx
    #> 2 orange       orange    264  6754 fruitdata4.xlsx
    #> 3 grapes       green     346  3423 fruitdata4.xlsx
    #> 4 strawberries red       456  5674 fruitdata4.xlsx

Use `here("data", .x)` on `file_list` on the way in. Now you’ll want to
use `basename()` on the paths before adding to the data frame.

    file_list %>% 
      here("data", .) %>% 
      map(~ read_excel(path = .x, range = "A1:D5")  %>%
            mutate("file_name" = basename(.x)))
    #> [[1]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    240  3635 fruitdata1.xlsx
    #> 2 orange       orange    375  3567 fruitdata1.xlsx
    #> 3 grapes       green     290  3564 fruitdata1.xlsx
    #> 4 strawberries red       187  3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    274  7578 fruitdata2.xlsx
    #> 2 orange       orange    363  8546 fruitdata2.xlsx
    #> 3 grapes       green     678  6478 fruitdata2.xlsx
    #> 4 strawberries red       343  4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    896  4345 fruitdata3.xlsx
    #> 2 orange       orange    453  2456 fruitdata3.xlsx
    #> 3 grapes       green     765  3564 fruitdata3.xlsx
    #> 4 strawberries red       345  5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    545  5684 fruitdata4.xlsx
    #> 2 orange       orange    264  6754 fruitdata4.xlsx
    #> 3 grapes       green     346  3423 fruitdata4.xlsx
    #> 4 strawberries red       456  5674 fruitdata4.xlsx

Use `list.files(full.names = TRUE)` when capturing the file paths.
Again, use `basename()` on the paths before adding to the data frame.

    file_list2 <- list.files(path = here("data"), pattern='*.xlsx', full.names = TRUE)
    file_list2
    #> [1] "/Users/jenny/tmp/map_excel_reprex/data/fruitdata1.xlsx"
    #> [2] "/Users/jenny/tmp/map_excel_reprex/data/fruitdata2.xlsx"
    #> [3] "/Users/jenny/tmp/map_excel_reprex/data/fruitdata3.xlsx"
    #> [4] "/Users/jenny/tmp/map_excel_reprex/data/fruitdata4.xlsx"

    file_list2 %>% 
      map(~ read_excel(path = .x, range = "A1:D5")  %>%
            mutate("file_name" = basename(.x)))
    #> [[1]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    240  3635 fruitdata1.xlsx
    #> 2 orange       orange    375  3567 fruitdata1.xlsx
    #> 3 grapes       green     290  3564 fruitdata1.xlsx
    #> 4 strawberries red       187  3367 fruitdata1.xlsx
    #> 
    #> [[2]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    274  7578 fruitdata2.xlsx
    #> 2 orange       orange    363  8546 fruitdata2.xlsx
    #> 3 grapes       green     678  6478 fruitdata2.xlsx
    #> 4 strawberries red       343  4674 fruitdata2.xlsx
    #> 
    #> [[3]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    896  4345 fruitdata3.xlsx
    #> 2 orange       orange    453  2456 fruitdata3.xlsx
    #> 3 grapes       green     765  3564 fruitdata3.xlsx
    #> 4 strawberries red       345  5647 fruitdata3.xlsx
    #> 
    #> [[4]]
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    545  5684 fruitdata4.xlsx
    #> 2 orange       orange    264  6754 fruitdata4.xlsx
    #> 3 grapes       green     346  3423 fruitdata4.xlsx
    #> 4 strawberries red       456  5674 fruitdata4.xlsx

Use `fs::dir_ls()` instead of `list.files()`, which IMO has better
defaults than `list.files()`. Now we’ll switch to `fs::path_file()` to
shorten the paths before adding to the data frame.

    library(fs)

    file_list_fs <- dir_ls(here("data"), glob = "*.xlsx")
    file_list_fs
    #> /Users/jenny/tmp/map_excel_reprex/data/fruitdata1.xlsx
    #> /Users/jenny/tmp/map_excel_reprex/data/fruitdata2.xlsx
    #> /Users/jenny/tmp/map_excel_reprex/data/fruitdata3.xlsx
    #> /Users/jenny/tmp/map_excel_reprex/data/fruitdata4.xlsx

    file_list_fs %>% 
      map(~ read_excel(path = .x, range = "A1:D5")  %>%
            mutate("file_name" = path_file(.x)))
    #> $`/Users/jenny/tmp/map_excel_reprex/data/fruitdata1.xlsx`
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    240  3635 fruitdata1.xlsx
    #> 2 orange       orange    375  3567 fruitdata1.xlsx
    #> 3 grapes       green     290  3564 fruitdata1.xlsx
    #> 4 strawberries red       187  3367 fruitdata1.xlsx
    #> 
    #> $`/Users/jenny/tmp/map_excel_reprex/data/fruitdata2.xlsx`
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    274  7578 fruitdata2.xlsx
    #> 2 orange       orange    363  8546 fruitdata2.xlsx
    #> 3 grapes       green     678  6478 fruitdata2.xlsx
    #> 4 strawberries red       343  4674 fruitdata2.xlsx
    #> 
    #> $`/Users/jenny/tmp/map_excel_reprex/data/fruitdata3.xlsx`
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    896  4345 fruitdata3.xlsx
    #> 2 orange       orange    453  2456 fruitdata3.xlsx
    #> 3 grapes       green     765  3564 fruitdata3.xlsx
    #> 4 strawberries red       345  5647 fruitdata3.xlsx
    #> 
    #> $`/Users/jenny/tmp/map_excel_reprex/data/fruitdata4.xlsx`
    #> # A tibble: 4 x 5
    #>   fruit        colour weight votes file_name      
    #>   <chr>        <chr>   <dbl> <dbl> <chr>          
    #> 1 banana       yellow    545  5684 fruitdata4.xlsx
    #> 2 orange       orange    264  6754 fruitdata4.xlsx
    #> 3 grapes       green     346  3423 fruitdata4.xlsx
    #> 4 strawberries red       456  5674 fruitdata4.xlsx

If `data` held *only* Excel files you could use `fs::dir_map()`. But it
won’t work as long as that `.Rmd` is in there.

    foo <- function(full_path) {
      read_excel(path = full_path, range = "A1:D5") %>%
        mutate("file_name" = path_file(full_path))
    }
    dir_map("data", foo)

I suspect you will eventually want to row bind these data frames? In
that case, I think the most pleasing work flow is with
`purrr::map_dfr()`. If your paths are well-named from the start, its
`id` argument accomplishes what you doing with `mutate()`.

    file_list_fs <- dir_ls(here("data"), glob = "*.xlsx")
    names(file_list_fs) <- path_file(file_list_fs)

    file_list_fs %>% 
      map_dfr(~ read_excel(path = .x, range = "A1:D5"), .id = "file_name")
    #> # A tibble: 16 x 5
    #>    file_name       fruit        colour weight votes
    #>  * <chr>           <chr>        <chr>   <dbl> <dbl>
    #>  1 fruitdata1.xlsx banana       yellow    240  3635
    #>  2 fruitdata1.xlsx orange       orange    375  3567
    #>  3 fruitdata1.xlsx grapes       green     290  3564
    #>  4 fruitdata1.xlsx strawberries red       187  3367
    #>  5 fruitdata2.xlsx banana       yellow    274  7578
    #>  6 fruitdata2.xlsx orange       orange    363  8546
    #>  7 fruitdata2.xlsx grapes       green     678  6478
    #>  8 fruitdata2.xlsx strawberries red       343  4674
    #>  9 fruitdata3.xlsx banana       yellow    896  4345
    #> 10 fruitdata3.xlsx orange       orange    453  2456
    #> 11 fruitdata3.xlsx grapes       green     765  3564
    #> 12 fruitdata3.xlsx strawberries red       345  5647
    #> 13 fruitdata4.xlsx banana       yellow    545  5684
    #> 14 fruitdata4.xlsx orange       orange    264  6754
    #> 15 fruitdata4.xlsx grapes       green     346  3423
    #> 16 fruitdata4.xlsx strawberries red       456  5674
