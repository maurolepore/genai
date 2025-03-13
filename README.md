

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
#> Hello. How can I help?
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
    #> with open("/tmp/Rtmp2z7zhr/file2416836234077", "w") as f:
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

writeLines(readLines(file))
#> hello world
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
#> I am sorry, I cannot fulfill your request. I do not have the ability to access 
#> or process the contents of files. Therefore, I cannot locate the table of 
#> contents or extract the content of chapter 6.
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

writeLines(readLines("authors-guide.md"))
#> 6 Guide for Authors
#> 50
#> 6.1 Planning a Submission (or a Pre-Submission Enquiry)
#> 50
#> 6.2 Preparing for Submission
#> 51
#> 6.3 The Submission Process
#> 51
#> 6.4 The Review Process .
#> 52

# Clean up
unlink("authors-guide.md")
```
