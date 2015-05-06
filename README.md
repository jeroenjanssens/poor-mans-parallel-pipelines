%title: Poor Man's Parallel Pipelines
%author: Jeroen Janssens - @jeroenhjanssens
%date: 2015-05-07



-> A brief introduction to <-

\          \_\_\_\_\_ \_   \_ \_    \_   \_\_\_\_\_                \_ \_      \_ 
\         / \_\_\_\_| \\ | | |  | | |  \_\_ \\              | | |    | |
\        | |  \_\_|  \\| | |  | | | |\_\_) |\_ \_ \_ \_\_ \_\_ \_| | | \_\_\_| |
\        | | |\_ | . \` | |  | | |  \_\_\_/ \_\` | '\_\_/ \_\` | | |/ \_ \\ |
\        | |\_\_| | |\\  | |\_\_| | | |  | (\_| | | | (\_| | | |  \_\_/ |
\         \\\_\_\_\_\_|\_| \\\_|\\\_\_\_\_/  |\_|   \\\_\_,\_|\_|  \\\_\_,\_|\_|\_|\\\_\_\_|\_|

-> or <-

-> _*Poor Man's Parallel Pipelines*_ <-

-> Jeroen Janssens <-

-> 2015-05-07 <-


------------------------------------------------------------------------------

# Introduction

GNU `parallel`.


------------------------------------------------------------------------------

# What is GNU Parallel

- easy to set up
- small tool
- parallelize ordinary pipelines

------------------------------------------------------------------------------

# Examples 

- Calling an API multiple times
- Processing many files
- Split up stream to multiple processes
- Execute command on all your instances


------------------------------------------------------------------------------

# GNU Parallel is a glorified for loop


	$ for i in $(seq 5); do echo "Hi $i"; done
^
	Hi 1
	Hi 2
	Hi 3
	Hi 4
	Hi 5
^

	$ seq 5 | parallel "echo Hi {}"
^
	Hi 3
	Hi 2
	Hi 1
	Hi 4
	Hi 5


------------------------------------------------------------------------------

# Installing GNU Parallel


On Mac OS X:

	$ brew install parallel

On Ubuntu Linux:

	$ sudo apt-get install parallel

*Don't use the version provided by* `moreutils`

On Microsoft Windows:

	C:\>vagrant init data-science-toolbox/data-science-at-the-command-line


------------------------------------------------------------------------------

# Distributed data processing frameworks are great

- Hadoop
- Spark
- Samza
- Storm
- Flink

^
But a bit too cumbersome for one-off data science tasks


------------------------------------------------------------------------------

# Data Science at the Command Line

Published by O'Reilly in October 2014.

The [book's website](http://datascienceatthecommandline.com) contains:

- Instructions to set up the virtual machine
- Overview of all the command-line tools

50% discount code at O'Reilly: *TS2015*


------------------------------------------------------------------------------

# Difference with xargs

------------------------------------------------------------------------------

# Working with files

ls 


------------------------------------------------------------------------------

# Know your placeholders 





------------------------------------------------------------------------------

# Split standard input


------------------------------------------------------------------------------

# Calling The New York Times API

Cartesian product of year and page:

	$ parallel -j1 --progress --delay 0.1 --results results "curl -sL " \
^
	> "'http://api.nytimes.com/svc/search/v2/articlesearch.json?q=New+'"\
^
	> "'York+Fashion+Week&begin_date={1}0101&end_date={1}1231&page={2}'"\
^
	> "'&api-key=${KEY}'" ::: {2009..2013} ::: {0..99} > /dev/null
^
	
	Computers / CPU cores / Max jobs to run
	1:local / 4 / 1
	
	Computer:jobs running/jobs completed/%of started jobs/Average seconds to
	complete
	local:1/9/100%/0.4s


------------------------------------------------------------------------------

# Separate output

	$ tree results | head
	results
	`-- 1
		|-- 2009
		|   `-- 2
		|       |-- 0
		|       |   |-- stderr
		|       |   `-- stdout
		|       |-- 1
		|       |   |-- stderr
		|       |   `-- stdout

	$ cat results/1/*/2/*/stdout

------------------------------------------------------------------------------

# Multiple jobs  

------------------------------------------------------------------------------

# One job at a time

------------------------------------------------------------------------------

# Help!

	$ man parallel
^
	PARALLEL(1)                       parallel                       PARALLEL(1)
	
	
	
	NAME
		   parallel - build and execute shell command lines from standard input
		   in parallel
	
	SYNOPSIS
		   parallel [options] [command [arguments]] < list_of_arguments
	
		   parallel [options] [command [arguments]] ( ::: arguments | ::::
		   argfile(s) ) ...
	
		   parallel --semaphore [options] command

^
-> *More than 100 command-line options!* <-


------------------------------------------------------------------------------

# CSV as input


------------------------------------------------------------------------------
 
# Executing pipeline on remote instances


Installing GNU Parallel:


------------------------------------------------------------------------------

# Obtain list of EC2 instances 


------------------------------------------------------------------------------

# basefile?


------------------------------------------------------------------------------

# Resume jobs 


------------------------------------------------------------------------------

# R / Python


------------------------------------------------------------------------------

# Conclusion

- Come to the book signing session in the break
