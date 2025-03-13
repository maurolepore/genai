

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
#> Hello! How can I help you?
```

May lack tools for common jobs:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
file <- tempfile()

prompt <- paste("Write 'hello world' to", file)
chat$chat(prompt)
```

    #> ```python
    #> with open("/tmp/Rtmp4giAC2/file26a1a56bbea0e", "w") as f:
    #>     f.write("hello world")
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
  text = type_string("A character string.", required = TRUE),
  con = type_string("The path to the file to write.", required = TRUE)
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
#> Done.

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
  - Extract the table of contents for chapter 6: 'Guide for Authors'
  - Present it without page number in the format '1.1. Section Title'
  - Write it to authors-contents.md
)"
)
#> I have written '1.1. Section Title' to authors-contents.md.
```

Provide the necessary context:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$register_tool(file_writer)

chat$chat(
  content_pdf_url("https://devguide.ropensci.org/ropensci-dev-guide.pdf"),
  r"(
  - In the first 5 pages find the table of contents
  - Extract the table of contents for chapter 6: 'Guide for Authors'
  - Present it without page number in the format '1.1. Section Title'
  - Write it to authors-contents.md
  )"
)
#> Okay, I've written the table of contents for chapter 6 to 
#> `authors-contents.md`.

readLines("authors-contents.md")
#> [1] "6.1 Planning a Submission (or a Pre-Submission Enquiry)"
#> [2] "6.2 Preparing for Submission"                           
#> [3] "6.3 The Submission Process"                             
#> [4] "6.4 The Review Process"

# Clean up
unlink("authors-contents.md")
```

Text files are easy to read:

``` r
chat <- chat_openai(
  system_prompt = "You are a friendly but terse rOpenSci editor."
)
#> Using model = "gpt-4o".

chat$chat(
  "Is the ixplorer package in scope for rOpenSci software-review?",
  "# rOpenSci package categories in scope for sofware-review",
  readLines(
    "https://raw.githubusercontent.com/ropensci/dev_guide/refs/heads/main/softwarereview_policies.Rmd"
  ),
  "# DESCRIPTION of the ixplorer package",
  readLines(
    "https://raw.githubusercontent.com/ixpantia/ixplorer/refs/heads/master/DESCRIPTION"
  )
)
#> The `ixplorer` package seems to focus on creating and managing tickets in 
#> 'gitea' using R, which aligns with workflow automation under rOpenSci’s package
#> categories. However, the package is primarily a client for a specific service 
#> ('gitea'), and it would need to demonstrate how it supports reproducible 
#> research significantly beyond direct API access. You might want to provide a 
#> pre-submission inquiry on the rOpenSci GitHub, detailing how the package serves
#> scientific workflows and automates tasks in a reproducible manner, particularly
#> if there are enhancements over existing tools or integration with broader 
#> scientific data management processes.
```
