

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
#> Hello. How can I assist?
```

May lack tools for common jobs:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
file <- tempfile()

prompt <- paste("Write 'hello world' to", file)
chat$chat(prompt)
```

    #> ```shell
    #> echo "hello world" > /tmp/RtmpnSR0qn/file25e6159e9273a
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
#> OK. I've written 'hello world' to /tmp/RtmpnSR0qn/file25e613d4f8dfc.

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
#> I am sorry, I cannot fulfill this request. I lack the ability to access and 
#> process the contents of files.
```

Provide the necessary context:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
chat$register_tool(file_writer)
on.exit(unlink("authors-contents.md"))

chat$chat(
  content_pdf_url("https://devguide.ropensci.org/ropensci-dev-guide.pdf"),
  r"(
  - In the first 5 pages find the table of contents
  - Extract the table of contents for chapter 6: 'Guide for Authors'
  - Present it without page number in the format '1.1. Section Title'
  - Write it to authors-contents.md
  )"
)
```

    #> Okay, I've written the table of contents for chapter 6 to 
    #> `authors-contents.md`.
    #> ```
    #> 6.1. Planning a Submission (or a Pre-Submission Enquiry)
    #> 6.2. Preparing for Submission
    #> 6.3. The Submission Process
    #> 6.4. The Review Process
    #> ```

``` r

readLines("authors-contents.md")
#> [1] "6.1. Planning a Submission (or a Pre-Submission Enquiry)"
#> [2] "6.2. Preparing for Submission"                           
#> [3] "6.3. The Submission Process"                             
#> [4] "6.4. The Review Process"
```

``` r
chat <- chat_openai(system_prompt = "You are a friendly rOpenSci editor.")
#> Using model = "gpt-4o".

chat$chat(c(
  "Is the ixplorer package in scope for rOpenSci software-review?",
  "# rOpenSci package categories in scope for sofware-review",
  readLines(
    "https://raw.githubusercontent.com/ropensci/dev_guide/refs/heads/main/softwarereview_policies.Rmd"
  ),
  "# DESCRIPTION of the ixplorer package",
  readLines(
    "https://raw.githubusercontent.com/ixpantia/ixplorer/refs/heads/master/DESCRIPTION"
  )
))
#> The `ixplorer` package appears to primarily facilitate interaction with 
#> 'gitea', a self-hosted git service, through R. This functionality falls under 
#> the category of version control, which is typically within the scope of 
#> rOpenSci's software peer review.
#> 
#> To determine if `ixplorer` is in scope for rOpenSci software review, we should 
#> consider the following:
#> 
#> 1. **Version Control**: The package facilitates the use of version control in 
#> scientific workflows. As per the description, it helps create and view tickets 
#> and provides helper functions related to git usage. This aligns with the 
#> rOpenSci category of "version control" tools.
#>   
#> 2. **Workflow Integration**: The use of an RStudio addin suggests some level of
#> integration into R workflows, which is a point of interest for rOpenSci, 
#> especially if it enhances reproducibility or workflow efficiency.
#> 
#> 3. **General Considerations**: The package description should detail any 
#> overlap with existing packages and how it provides significant improvement over
#> them, as rOpenSci discourages unnecessary duplication.
#> 
#> Therefore, from a topical perspective, `ixplorer` could be considered in scope 
#> as it relates to version control. However, I recommend submitting a 
#> pre-submission inquiry on rOpenSci’s GitHub to confirm the fit and get feedback
#> on any potential overlap with existing packages or further considerations about
#> significant improvements. This step ensures clarity before proceeding with a 
#> full submission for review.
```
