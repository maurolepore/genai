

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
#> Hello.
```

May lack tools for common jobs:

``` r
chat <- chat_gemini(system_prompt = "You are a friendly but terse assistant.")
#> Using model = "gemini-2.0-flash".
file <- tempfile()

prompt <- paste("Write 'hello world' to", file)
chat$chat(prompt)
```

    #> ```bash
    #> echo "hello world" > /tmp/Rtmpzo8HsN/file263a32f999de4
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
#> [1] "6.1. Planning a Submission (or a Pre-Submission Enquiry)"
#> [2] "6.2. Preparing for Submission"                           
#> [3] "6.3. The Submission Process"                             
#> [4] "6.4. The Review Process"

# Clean up
unlink("authors-contents.md")
```

Text files are easy to read:

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
#> To determine whether the `ixplorer` package is in scope for rOpenSci's software
#> review, we need to examine the package's functionality and compare it against 
#> the categories outlined in the rOpenSci Aims and Scope section.
#> 
#> The `ixplorer` package provides tools for creating and viewing tickets in 
#> 'gitea' (a self-hosted git service) using an RStudio addin, as well as 
#> providing helper functions for publishing documentation and using git. This 
#> suggests that the package is focused on workflow and collaboration tools, 
#> particularly for managing documentation and potentially aiding in version 
#> control.
#> 
#> Let's review whether this aligns with rOpenSci categories:
#> 
#> 1. **Workflow Automation**: The package might fit here if it automates and 
#> links workflows through RStudio addins, though it seems more focused on ticket 
#> management rather than a broader automation of workflows like those that 
#> integrate and manage workflows end-to-end.
#> 
#> 2. **Version Control**: The package has some functions related to using git. 
#> However, it primarily interacts with Gitea for ticket management rather than 
#> version control processes themselves. rOpenSci's criteria for version control 
#> tools emphasize facilitating scientific workflow management, which may or may 
#> not align closely depending on the depth of functionality related to git.
#> 
#> 3. **Scientific Software Wrappers**: The package wraps functionality for Gitea,
#> which is a utility for software project hosting and collaboration. While it 
#> interfaces with software utilities, the emphasis is not on scientific research 
#> tools but more on project management and collaboration.
#> 
#> Based on these observations, it seems that the core focus of the package aligns
#> more closely with project management and ticketing when using Gitea, rather 
#> than directly enhancing scientific reproducible research practices or managing 
#> the data lifecycle. Therefore, the `ixplorer` package might not be directly in 
#> scope under the current rOpenSci categories unless it places stronger emphasis 
#> on scientific workflow management or adds significant features that directly 
#> facilitate scientific research processes.
#> 
#> If you still believe the package aligns with rOpenSci goals or has features 
#> justifying inclusion, you could submit a pre-submission inquiry to discuss the 
#> unique features that might make it a candidate for review.
```
