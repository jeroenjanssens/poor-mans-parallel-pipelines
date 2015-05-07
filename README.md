%title: Poor Man's Parallel Pipelines
%author: Jeroen Janssens - @jeroenhjanssens
%date: 2015-05-07
%note: Run presentation with `mdp README.md` # https://github.com/visit1985/mdp



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


-> _http://bit.ly/strata-parallel_ <-

------------------------------------------------------------------------------

# Data Science at the Command Line


- Published by O'Reilly in October 2014
- GNU Parallel is discussed in Chapter 8
- Overview of all tools on _http://datascienceatthecommandline.com_
- Book signing session in the afternoon break
- 50% discount code during Strata London: *TS2015*


------------------------------------------------------------------------------

# Outline


- Situation
- What is GNU Parallel?
- Installing GNU Parallel
- Usage
- Parallelizing your Python or R code
- Conclusion


------------------------------------------------------------------------------

# Situation


You have a one-off data science task that could use some speeding up

For example:

- Make many API calls
- Try various settings a machine learning algorithm
- Process many data sets
- Distribute click stream to multiple processes
- Get statistics from all your remote instances


------------------------------------------------------------------------------

# Distributed data processing frameworks are great


- Hadoop
- Spark
- Samza
- Storm
- Flink

However:

- Perhaps not worthwhile to set up
- Not always straightforward how to translate your code
- The tools you use may be a black box

------------------------------------------------------------------------------

# What is GNU Parallel?


- Unix command-line tool created by Ole Tange
- Easy to install and configure
- Parallelize and distribute ordinary pipelines
- Loops over files, lines, arguments, hosts
- Assumption: task is splittable

------------------------------------------------------------------------------

# Installing GNU Parallel


On Mac OS X:

	$ brew install parallel
^

On Ubuntu Linux:

	$ sudo apt-get install parallel

*Don't use the version provided by* `moreutils`
^

On Microsoft Windows:

	C:\>vagrant init data-science-toolbox/data-science-at-the-command-line

------------------------------------------------------------------------------

# GNU Parallel is a for loop on steroids


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

# Usage: Working with files


	$ ls logs/*.log | parallel "grep ERROR {}"


------------------------------------------------------------------------------

# Usage: Know your placeholders


Assume line is "/home/jeroen/data.csv"

- `{}`   : /home/jeroen/data.csv
- `{.}`  : /home/jeroen/data
- `{/}`  : data.csv
- `{/.}` : data

Example:

	$ echo '/home/jeroen/data.csv' | parallel --dryrun "tar -czf {/.}.tar.gz {}"
	tar -czvf data.tar.gz /home/jeroen/data.csv

Other placeholders:

- `{1}` : first argument
- `{#}` : job number

------------------------------------------------------------------------------

# Usage: Named placeholders from CSV file


Use `head` and `csvlook` (from csvkit) to inspect data set:

	$ head -n 6 tips.csv | csvlook
	|--------+------+--------+--------+-----+--------+-------|
	|  bill  | tip  | sex    | smoker | day | time   | size  |
	|--------+------+--------+--------+-----+--------+-------|
	|  16.99 | 1.01 | Female | No     | Sun | Dinner | 2     |
	|  10.34 | 1.66 | Male   | No     | Sun | Dinner | 3     |
	|  21.01 | 3.5  | Male   | No     | Sun | Dinner | 3     |
	|  23.68 | 3.31 | Male   | No     | Sun | Dinner | 2     |
	|  24.59 | 3.61 | Female | No     | Sun | Dinner | 4     |
	|--------+------+--------+--------+-----+--------+-------|
^

	$ head -n 6 tips.csv | parallel -C, --header : \
	> "echo On {day}, {size} people had {time} for a total of '$'{bill}"
	On Sun, 2 people had Dinner for a total of $16.99
	On Sun, 3 people had Dinner for a total of $10.34
	On Sun, 3 people had Dinner for a total of $21.01
	On Sun, 2 people had Dinner for a total of $23.68
	On Sun, 4 people had Dinner for a total of $24.59


------------------------------------------------------------------------------

# Usage: Specifying number of concurrent jobs  


	$ cat input | parallel --jobs 10

Twice the number of cores:

	$ cat input | parallel --jobs 200%

Sometimes you only want one job at a time

	$ cat input | parallel -j1

------------------------------------------------------------------------------

# Example: Calling The New York Times API


Cartesian product of year and page:

	$ parallel -j1 --progress --delay 0.1 --results results "curl -sL " \
	> "'http://api.nytimes.com/svc/search/v2/articlesearch.json?q=New+'"\
	> "'York+Fashion+Week&begin_date={1}0101&end_date={1}1231&page={2}'"\
	> "'&api-key=${KEY}'" ::: {2009..2013} ::: {0..99} > /dev/null
^
	
	Computers / CPU cores / Max jobs to run
	1:local / 4 / 1
	
	Computer:jobs running/jobs completed/%of started jobs/Average seconds to
	complete
	local:1/9/100%/0.4s

------------------------------------------------------------------------------

# Example: Calling The New York Times API


Structured output:

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

Combine output using `cat`:

	$ cat results/1/*/2/*/stdout

------------------------------------------------------------------------------

# Aside: Obtain list of EC2 instances 


	$ aws ec2 describe-instances | jq '.Reservations[].Instances[] | '\
	> '{public_dns: .PublicDnsName, state: .State.Name}'
	{
	  "state": "running",
	  "public_dns": "ec2-54-88-122-140.compute-1.amazonaws.com"
	}
	{
	  "state": "stopped",
	  "public_dns": null
	}


	$ aws ec2 describe-instances | jq -r '.Reservations[].Instances[] | '\
	> 'select(.State.Name=="running") | .PublicDnsName' > instances
	$ cat instances
	ec2-54-88-122-140.compute-1.amazonaws.com
	ec2-54-88-89-208.compute-1.amazonaws.com


------------------------------------------------------------------------------
 
# Usage: Executing pipeline on remote instances


Get hostname of all instances:

	$ parallel --nonall --sshloginfile instances hostname
	ip-172-31-23-204
	ip-172-31-23-205

Install GNU Parallel:

	$ parallel --nonall --slf instances "sudo apt-get install -y parallel"

------------------------------------------------------------------------------

# Parallelizing your Python or R code

Recipe to create reusable command-line tool:


1. Copy and paste code into a file
2. Add execute permissions using `chmod`
3. Define a so-called shebang
4. Remove the fixed input part
5. Add a parameter
6. Optionally extend your PATH


------------------------------------------------------------------------------

# Parallelizing your Python or R code


Top n words in Python:

	#!/usr/bin/env python
	import re
	import sys
	from collections import Counter
	num_words = int(sys.argv[1])
	text = sys.stdin.read().lower()
	words = re.split('\W+', text)
	cnt = Counter(words)
	for word, count in cnt.most_common(num_words):
		print "%7d %s" % (count, word)

^
	$ cat data/76.txt | ./top-words.py 3
	   6441 and
	   5082 the
	   3666 i

^
	$ ls data/*.txt | parallel --tag "cat {} | ./top-words.py 10"


------------------------------------------------------------------------------

# Parallelizing your Python or R code


Top n words in R:

	#!/usr/bin/env Rscript
	n <- as.integer(commandArgs(trailingOnly = TRUE))
	f <- file("stdin")
	lines <- readLines(f)
	words <- tolower(unlist(strsplit(lines, "\\W+")))
	counts <- sort(table(words), decreasing = TRUE)
	counts_n <- counts[1:n]
	cat(sprintf("%7d %s\n", counts_n, names(counts_n)), sep = "")
	close(f)

^
	$ cat data/76.txt | ./top-words.R 3
	   6441 and
	   5082 the
	   3666 i

^
	$ ls data/*.txt | parallel --tag "cat {} | ./top-words.R 10"

------------------------------------------------------------------------------

# Making sure your code is on the remote instances


Custom command-line tool that outputs sum of numbers:

	$ cat sum
	#!/usr/bin/env bash
	paste -sd+ | bc
^

	$ chmod u+x sum
^

	$ seq 3
	1
	2
	3
	$ seq 3 | ./sum
	6

^
	$ seq 1000 | parallel -N100 --basefile sum --pipe --slf instances './sum' | 
	> ./sum
	500500


------------------------------------------------------------------------------

# Usage: Split standard input

------------------------------------------------------------------------------

# Conclusion


- GNU Parallel is easy to install and setup
- Has a bit of a learning curve
- Handful of options cover 90% of use cases
- Can speed up ordinary pipelines and code
^

- Book signing session in the afternoon break
- 50% discount code during Strata London: *TS2015*


------------------------------------------------------------------------------

------------------------------------------------------------------------------

# Differences between GNU parallel and xargs

`xargs` offers similar functionality. However:

- `xargs` deals badly with special characters (such as space, ' and ")
- `xargs` has no support for running number-of-cpu-cores jobs in parallel
- `xargs` has no support for grouping the output
- `xargs` has no support for keeping the order of the output
- `xargs` has no support for running jobs on remote computers

Luckily, `parallel` is often a drop in replacement for `xargs`

