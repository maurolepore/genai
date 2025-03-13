

# genai

The goal of this repo is to show key features of the ellmer package for
generative AI.

Setup

``` r
library(ellmer)
```

Test.

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$chat("Hi")
#> Hello!
```

May lack tools for common jobs:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
file <- tempfile()

prompt <- paste("Write 'hello world' to", file)
chat$chat(prompt)
```

    #> ```tool_code
    #> echo 'hello world' > /tmp/RtmpgzvcfL/file24b4965ad498c
    #> ```

``` r

file.exists(file)
#> [1] FALSE
```

Create the necesasary tools:

``` r
# ellmer::create_tool_def(base::writeLines)
file_writer <- tool(
  base::writeLines,
  "Writes text to a file.",
  text = type_string(
    "A character string.",
    required = TRUE
  ),
  con = type_string(
    "The path to the file to write.",
    required = TRUE
  )
)
```

Provide the necessary tools:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$register_tool(file_writer)

file <- tempfile()
prompt <- paste("Write 'hello world' to", file)
chat$chat(prompt)
#> OK.

readLines(file)
#> [1] "hello world"
```

May lack the context you care about:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$register_tool(file_writer)

chat$chat(
  r"(
  - In the first 5 pages find the table of contents
  - Show me the contents of chapter 6: 'Guide for Authors'
  - Write it to authors-guide.md
)"
)
#> I am sorry, I cannot fulfill this request. I do not have the ability to access 
#> or read the contents of files. Therefore, I cannot locate the table of contents
#> or the content of chapter 6.
```

Provide the necessary context:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$register_tool(file_writer)
on.exit(unlink("authors-guide.md"))

chat$chat(
  content_pdf_url("https://devguide.ropensci.org/ropensci-dev-guide.pdf"),
  r"(
  - In the first 5 pages find the table of contents
  - Show me the contents of chapter 6: 'Guide for Authors'
  - Write it to authors-guide.md
  )"
)
#> I have written the contents of chapter 6 to authors-guide.md.

readLines("authors-guide.md")
#>  [1] "6 Guide for Authors"                                    
#>  [2] "50"                                                     
#>  [3] "6.1 Planning a Submission (or a Pre-Submission Enquiry)"
#>  [4] "50"                                                     
#>  [5] "6.2 Preparing for Submission"                           
#>  [6] "51"                                                     
#>  [7] "6.3 The Submission Process"                             
#>  [8] "51"                                                     
#>  [9] "6.4 The Review Process."                                
#> [10] "52"

# Clean up
unlink("authors-guide.md")
```
