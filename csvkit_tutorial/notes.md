# Adjust csv data and insert into a text document 
## Insert a table
Insert a table from "stacked.csv" into document in vim:\
`.r ! csvlook stacked.csv`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data1-test.csv |      1 |   2 |  20 |
| data1-test.csv |      2 |  13 |  34 |
| data2-test.csv |      1 |   3 | 205 |
| data2-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |
| data3-test.csv |      2 |  13 |  34 |
| data3-test.csv |      3 |  13 |  34 |
| data3-test.csv |      4 |  13 |  34 |

## Insert only selected columns of table
To insert only "group" and "peakNo" columns.

`.r ! csvcut -c group,peakNo | csvlook`

| group          | peakNo |
| -------------- | ------ |
| data1-test.csv |      1 |
| data1-test.csv |      2 |
| data2-test.csv |      1 |
| data2-test.csv |      2 |
| data2-test.csv |      3 |
| data2-test.csv |      4 |
| data3-test.csv |      2 |
| data3-test.csv |      3 |
| data3-test.csv |      4 |

## Insert table sorted by one or multiple columns
To insert a table sorted by two columns (max and peakNo):\
`.r ! csvsort -c max,peakNo stacked.csv | csvlook`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data1-test.csv |      1 |   2 |  20 |
| data1-test.csv |      2 |  13 |  34 |
| data2-test.csv |      2 |  13 |  34 |
| data3-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data3-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |
| data3-test.csv |      4 |  13 |  34 |
| data2-test.csv |      1 |   3 | 205 |

\-r flag: revert the sorting to descending

## Insert table filtered by word in given column
To select only rows containing "data2" use -m flag:\
`.r ! csvgrep -c group -m data2 stacked.csv | csvlook`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data2-test.csv |      1 |   3 | 205 |
| data2-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |

\-i flag: to revert from select matching to return not matching

## Insert table filtered by regular expression in given column
To filter rows which in group column contain "ta2" or "ta3" use -r flag:\
`.r ! csvgrep -c group -r "ta2|ta3" stacked.csv | csvlook`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data2-test.csv |      1 |   3 | 205 |
| data2-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |
| data3-test.csv |      2 |  13 |  34 |
| data3-test.csv |      3 |  13 |  34 |
| data3-test.csv |      4 |  13 |  34 |

\-i flag: to revert from select matching to return not matching

## Filter table rows by matching list contained in a file
### Cell in csv is matched exactly by line
To filter by file "filter-file" which contains 2 lines: data3-test.csv and data1-test.csv.\

**Print the lines containing what is in the file**\

`.r ! csvgrep -c group -f filter-file stacked.csv | csvlook`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data1-test.csv |      1 |   2 |  20 |
| data1-test.csv |      2 |  13 |  34 |
| data3-test.csv |      2 |  13 |  34 |
| data3-test.csv |      3 |  13 |  34 |
| data3-test.csv |      4 |  13 |  34 |

**print lines not containing what is in the file**\
`.r ! csvgrep -c group -if filter-file stacked.csv | csvlook`

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data2-test.csv |      1 |   3 | 205 |
| data2-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |

### Cell in csv contains string found in file
To filter by file which contains two lines: data3 and data1\
`.r ! cat stacked.csv | grep -Fvwf filter-file | csvlook`

grep flags:\
\-f: take from file\
\-F: as fixed strings\
\-W: only lines with whole words matches\
\-v: select non matching lines (filtering)\

| group          | peakNo | min | max |
| -------------- | ------ | --- | --- |
| data2-test.csv |      1 |   3 | 205 |
| data2-test.csv |      2 |  13 |  34 |
| data2-test.csv |      3 |  13 |  34 |
| data2-test.csv |      4 |  13 |  34 |

# Stack first lines of csvs together

## csvstack approach 
1. stack the csvs --filename flag creates a column with filenames\
`csvstack --filenames data*t.csv > stacked.csv`

2. select the (number of peaks) you want to keep\
`csvgrep -c peakNo -r "^[1-2]$" stacked.csv > filtered.csv`

Or this is an alternative\ 
`csvgrep -c peakNo -r "^1|2$" stacked.csv > filtered.csv`

3. Look at the data\
`csvlook filtered.csv`

4. Do a summary statistics\
`csvstat filtered.csv`

## Alternative grep approach
### Get the first 2 rows 
Use grep to search for rownumbers 1 and 2 in first column, pipe it into file

for file in <your-data-files>; do (csvgrep -c 1 -r"^1|2$" "${file}" >> first-two-rows.csv ); done 

This also inserts headers on top of each values

### Remove the repeating headers
Grep for pattern from first column header and exclude them 

csvgrep -c 1 -ir "pattern" first-two-rows.csv > headers-removed.csv

This works, the only trouble is I would like to have a column with a file name there and not use awk to do it.





# Learn how to use the csv kit
## Resources
[link to tutorial page](https://csvkit.readthedocs.io/en/latest/tutorial.html)\
[link to references and full manual for functions](https://csvkit.readthedocs.io/en/latest/cli.html)

## 1.2 installing
sudo pip install csvkit

This worked, but also recomend using virtualenv to keep the python packages in better order.

## 1.3 Getting the data
mkdir csvkit_tutorial
cd csvkit_tutorial

curl -L -O https://raw.githubusercontent.com/wireservice/csvkit/master/examples/realdata/ne_1033_data.xlsx

## 1.4 in2csv the excel killer
*transform to csv and show in terminal*\
in2csv ne_1033_data.xlsx

*transform to csv and save as data.csv*\
in2csv ne_1033_data.xlsx > data.csv

## 1.5 csvlook: dataperiscope

*to get the rough idea of the data use csvlook*\
csvlook data.csv

*to get it paged, separated in columns and able to navigate through, use less*\
csvlook data.csv | less -S

## 1.6 csvcut: datascalpel
*to select and reorder columns*\
* to display the numbers and names of the columns use -n flag*\
csvcut -n data.csv

*to display just columns of given numbers use `-c flag`*\
csvcut -c 2,5,6 data.csv

*to see the selected columns in the nice less -S format combine with csvlook*\

csvcut -c 2,5,6 data.csv | csvlook | less -S

*csvcut also understands the names of the columns as long as there are no spaces after commas*\
*the commas are tricky, if in the name double quote (even in beginning in the column name)*\
csvcut -c county,item_name,quantity data.csv

The question here is, can I use the csvcut to also filter the lines, so for example search for the first and second
item-ie peak?\
I have an idea using the csvgrep and using "^1$" in the first column, but there maybe a better high level command to
achieve this.\
But looks there is nothing in here, so lets try this on my own.\

## 1.7 connecting commads through pipes
easy, not much to add here

# 2. Examining the data
## 2.1 csvstat: statistics without code
*inspired by the summary from, pick columns and and pipe to stat*\
csvcut -c county,acquisition_cost,ship_date data.csv | csvstat

## csvgrep: find the data you need
*csvgrep with -c flag sets columns where to search, -m flag what to search*\
csvcut -c county,item_name,total_cost data.csv | csvgrep -c county -m LANCASTER | csvlook

*the -r flag gives you the chance to go for regular expressions"*\
*in case of -r flag the pattern needs to be doublequoted*\
*case insensitivity is achieved by "(?i)pattern"*\

csvgrep -c 2 -r "(?i)lancaster" data.csv | csvcut -c 1,2,5 | csvlook | less -S

*The insensitivity flag applies to all the expressions afterwards*\
csvgrep -c 5 -r "(?i)ri|li" data.csv | csvcut -c 1,2,5 | csvlook | less -S

## 2.3 csvsort: order matters
*there is an utility to do the sorting*\
To sort the rows by the `total_cost` column in descending order use the `-r` flag:\

csvcut -c county,item_name,total_cost data.csv | csvgrep -c county -m LANCASTER | csvsort -c total_cost -r | csvlook

there is a question here: What commands would you use to figure out whether other counties also recieved large number of
vehicles?\
No idea, so lets break it down:

1. find the vehicles: using `csvgrep -c item_name -r "(?i)vehicle"`
2. Sort by the number of vehicles in the county, but I guess this would come in the next part of the tutorial.

first see if I can see how many items they recieved:\
csvcut -n data.csv

Yes I can, there is a column called quantity.\
Next to get the quantity of items in data and see the top\
csvcut -c county,item_name,quantity data.csv | head |csvlook| less -S

This also works, so now I want to restrict the items to vehicles only
csvcut -c county,item_name,quantity data.csv | csvgrep -c item_name -r "(?i)vehicle" |head |csvlook| less -S

This finally works and we can see that only lancaster county recieved some vehicles.

# 3. Power tools
## 3.1 csvjoin: merging related data
