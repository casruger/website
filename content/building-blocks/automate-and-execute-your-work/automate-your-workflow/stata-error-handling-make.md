---
title: "Integrating Stata in Automated Workflows"
description: "Learn how to use R to check for errors and the completion of Stata code in batch mode or in a Makefile."
keywords: "exception, handling, stata, R, make, error"
#date: 2021-03-19
#weight: 2
author: "Nazli Alagöz"
authorlink: "https://www.tilburguniversity.edu/staff/n-m-alagoz"
aliases:
 - /error-handling/stata-make
 - /automate/stata
 - /building-blocks/automate-and-execute-your-work/error-handling/stata-error-handling-make/
---

## Overview <!-- Goal of the Building Block -->

When you run Stata within an automated research pipeline (e.g., using a `makefile`), Stata does *not stop the progression* of the Makefile, even if there is an error in your cod! Thus, you have no idea whether Stata code was executed without any errors until the end without checking the Stata log files.

To remedy this issue, you can use R to check for any error that may have occurred in the log file. If there was an error, we can make the workflow to interrupt.

## Code

We show you how to very simply check the Stata log files for errors and stop the Makefile if there are any errors.

- First off, make sure you have [Make set up](/get/make), [Stata](/get/stata) and [R](/get/r/) executables are added to your environment variable called "PATH".
- Also make sure that your Stata do-file produces a log file.

Next, we create an R script called `logcheck.R` that checks for errors and the completion of the do-file from the log file.

### Create an R script that checks for errors and completion of the do-file

{{% codeblock %}}
```R
# Define the arguments
args = commandArgs(trailingOnly=TRUE)

# Test if there is at least one argument: if not, return an error
if (length(args)==0) {
  stop("At least one argument must be supplied (input file).n", call.=FALSE)
}

# Read the log file
filecontent = readLines(args)

# Check if the log file includes an error massage and if so stop and display an error message
if (any(grepl('^r[(][0-9]',filecontent,ignore.case=T))) {
 stop(paste0('Log file for ', args, ' contains errors!'))
}
# Check whether the do-file was executed completely
if (!any(grepl('(end of do-file)',filecontent,ignore.case=T))) {

  stop(paste0('File (', args, ') has not been processed entirely!'))
}
# If no errors, report that there are no errors.
cat(paste0('Log file for ', args, ' checked. No errors.'))
```
{{% /codeblock %}}

### Incorporate the check in the Makefile
Here, you can replicate any rule where you run a do-file which creates a log file. We just use some random rule:

{{% codeblock %}}

```bash
# Define a rule where you use a do-file
target_file: prerequisite.do # define your target and prerequisites
	rm prerequisite.log # remove older log file produced by prerequisite.do previously
	StataMP-64.exe -e do prerequisite.do # execute the do-file
	Rscript logcheck.R prerequisite.log # check the log file for errors or incompletion
```

{{% /codeblock %}}
