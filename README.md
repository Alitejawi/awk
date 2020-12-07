<p align="center">
  <a href="https://github.com/Alitejawi/Clean-macOS-Setup">
 <img width=75px src="https://raw.githubusercontent.com/file-icons/icons/master/svg/Awk.svg"></a>
</p>

<h2 align="center">Awk</h2>

<div align="center">

[![Status](https://img.shields.io/github/last-commit/Alitejawi/awk.svg?style=flat-square)](https://github.com/Alitejawi/awk/commits/master)
[![GitHub Issues](https://img.shields.io/github/issues/Alitejawi/awk.svg?style=flat-square)](https://github.com/Alitejawi/awk/issues)
[![License](https://img.shields.io/github/license/Alitejawi/awk?style=flat-square)](https://github.com/Alitejawi/awk/blob/master/LICENSE)

</div>

---

<p align="center">
💻 A collection of awk snippets, see Learnbyexample's Github page for more. I have gathered these for easier access. 
  <br>
</p>

## <a name="field-processing"></a>Field processing

<br>

#### <a name="default-field-separation"></a>Default field separation

* `$0` contains the entire input record
    * default input record separator is newline character
* `$1` contains the first field text
    * default input field separator is one or more of continuous space, tab or newline characters
* `$2` contains the second field text and so on
* `$(2+3)` result of expressions can be used, this one evaluates to `$5` and hence gives fifth field
    * similarly if variable `i` has value `2`, then `$(i+3)` will give fifth field
    * See also [gawk manual - Expressions](https://www.gnu.org/software/gawk/manual/html_node/Expressions.html)
* `NF` is a built-in variable which contains number of fields in the current record
    * so, `$NF` will give last field
    * `$(NF-1)` will give second last field and so on

```bash
$ cat fruits.txt
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field
$ awk '{print $1}' fruits.txt
fruit
apple
banana
fig
guava

$ # print only second field
$ awk '{print $2}' fruits.txt
qty
42
31
90
6
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option
* by setting `FS` variable
* See [FPAT and FIELDWIDTHS](#fpat-and-fieldwidths) section for other ways of defining input fields

```bash
$ # second field where input field separator is :
$ echo 'foo:123:bar:789' | awk -F: '{print $2}'
123

$ # last field
$ echo 'foo:123:bar:789' | awk -F: '{print $NF}'
789

$ # first and last field
$ # note the use of , and space between output fields
$ echo 'foo:123:bar:789' | awk -F: '{print $1, $NF}'
foo 789

$ # second last field
$ echo 'foo:123:bar:789' | awk -F: '{print $(NF-1)}'
bar

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | awk -F';' '{print $3}'
three
```

* Regular expressions based input field separator

```bash
$ echo 'Sample123string54with908numbers' | awk -F'[0-9]+' '{print $2}'
string

$ # first field will be empty as there is nothing before '{'
$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $1}'

$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $2}'
foo
$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $3}'
bar
```

* default input field separator is one or more of continuous space, tab or newline characters (will be termed as whitespace here on)
    * exact same behavior if `FS` is assigned single space character
* in addition, leading and trailing whitespaces won't be considered when splitting the input record

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b     c
$ printf ' a    ate b\tc   \n' | awk '{print $1}'
a
$ printf ' a    ate b\tc   \n' | awk '{print NF}'
4
$ # same behavior if FS is assigned to single space character
$ printf ' a    ate b\tc   \n' | awk -F' ' '{print $1}'
a
$ printf ' a    ate b\tc   \n' | awk -F' ' '{print NF}'
4

$ # for anything else, leading/trailing whitespaces will be considered
$ printf ' a    ate b\tc   \n' | awk -F'[ \t]+' '{print $2}'
a
$ printf ' a    ate b\tc   \n' | awk -F'[ \t]+' '{print NF}'
6
```

* assigning empty string to FS will split the input record character wise
* note the use of command line option `-v` to set FS

```bash
$ echo 'apple' | awk -v FS= '{print $1}'
a
$ echo 'apple' | awk -v FS= '{print $2}'
p
$ echo 'apple' | awk -v FS= '{print $NF}'
e

$ # detecting multibyte characters depends on locale
$ printf 'hi👍 how are you?' | awk -v FS= '{print $3}'
👍
```

**Further Reading**

* [gawk manual - Field Splitting Summary](https://www.gnu.org/software/gawk/manual/html_node/Field-Splitting-Summary.html#Field-Splitting-Summary)
* [stackoverflow - explanation on default FS](https://stackoverflow.com/questions/30405694/default-field-separator-for-awk)
* [unix.stackexchange - filter lines if it contains a particular character only once](https://unix.stackexchange.com/questions/362550/how-to-remove-line-if-it-contains-a-character-exactly-once)
* [stackoverflow - Processing 2 files with different field separators](https://stackoverflow.com/questions/24516141/awk-processing-2-files-with-different-field-separators)

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* by setting `OFS` variable
* also gets added between every argument to `print` statement
    * use [printf](#printf-formatting) to avoid this
* default is single space

```bash
$ # statements inside BEGIN are executed before processing any input text
$ echo 'foo:123:bar:789' | awk 'BEGIN{FS=OFS=":"} {print $1, $NF}'
foo:789
$ # can also be set using command line option -v
$ echo 'foo:123:bar:789' | awk -F: -v OFS=':' '{print $1, $NF}'
foo:789

$ # changing a field will re-build contents of $0
$ echo ' a      ate b   ' | awk '{$2 = "foo"; print $0}' | cat -A
a foo b$

$ # $1=$1 is an idiomatic way to re-build when there is nothing else to change
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{print $0}'
foo:123:bar:789
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{$1=$1; print $0}'
foo-123-bar-789

$ # OFS is used to separate different arguments given to print
$ echo 'foo:123:bar:789' | awk -F: -v OFS='\t' '{print $1, $3}'
foo     bar

$ echo 'Sample123string54with908numbers' | awk -F'[0-9]+' '{$1=$1; print $0}'
Sample string with numbers
```

## Field processing

As mentioned before, `awk` is primarily used for field based processing. Consider the sample input file shown below with fields separated by a single space character.

>The [learn_gnuawk repo](https://github.com/learnbyexample/learn_gnuawk/tree/master/example_files) has all the files used in examples.

```bash
$ cat table.txt
brown bread mat hair 42
blue cake mug shirt -7
yellow banana window shoes 3.14
```

Here are some examples that are based on specific field rather than entire line. By default, `awk` splits the input line based on spaces and the field contents can be accessed using `$N` where `N` is the field number required. A special variable `NF` is updated with the total number of fields for each input line. There's more details to cover, but for now this is enough to proceed.

```bash
$ # print the second field of each input line
$ awk '{print $2}' table.txt
bread
cake
banana

$ # print lines only if last field is a negative number
$ # recall that default action is to print the contents of $0
$ awk '$NF<0' table.txt
blue cake mug shirt -7

$ # change 'b' to 'B' only for the first field
$ awk '{gsub(/b/, "B", $1)} 1' table.txt
Brown bread mat hair 42
Blue cake mug shirt -7
yellow banana window shoes 3.14
```


## Strings and Numbers

Some examples so far have already used string and numeric literals. As mentioned earlier, `awk` tries to provide a concise way to construct a solution from the command line. The data type of a value is determined based on the syntax used. String literals are represented inside double quotes. Numbers can be integers or floating point. Scientific notation is allowed as well. See [gawk manual: Constant Expressions](https://www.gnu.org/software/gawk/manual/gawk.html#Constants) for more details.

```bash
$ # BEGIN{} is also useful to write awk program without any external input
$ awk 'BEGIN{print "hi"}'
hi

$ awk 'BEGIN{print 42}'
42
$ awk 'BEGIN{print 3.14}'
3.14
$ awk 'BEGIN{print 34.23e4}'
342300
```

You can also save these literals in variables and use it later. Some variables are predefined, for example `NF`.

```bash
$ awk 'BEGIN{a=5; b=2.5; print a+b}'
7.5

$ # strings placed next to each other are concatenated
$ awk 'BEGIN{s1="con"; s2="cat"; print s1 s2}'
concat
```

If uninitialized variable is used, it will act as empty string in string context and `0` in numeric context. You can force a string to behave as a number by simply using it in an expression with numeric values. You can also use unary `+` or `-` operators. If the string doesn't start with a valid number (ignoring any starting whitespaces), it will be treated as `0`. Similarly, concatenating a string to a number will automatically change the number to string. See [gawk manual: How awk Converts Between Strings and Numbers](https://www.gnu.org/software/gawk/manual/gawk.html#Strings-And-Numbers) for more details.

```bash
$ # same as: awk 'BEGIN{sum=0} {sum += $NF} END{print sum}'
$ awk '{sum += $NF} END{print sum}' table.txt
38.14

$ awk 'BEGIN{n1="5.0"; n2=5; if(n1==n2) print "equal"}'
$ awk 'BEGIN{n1="5.0"; n2=5; if(+n1==n2) print "equal"}'
equal
$ awk 'BEGIN{n1="5.0"; n2=5; if(n1==n2".0") print "equal"}'
equal

$ awk 'BEGIN{print 5 + "abc 2 xyz"}'
5
$ awk 'BEGIN{print 5 + " \t 2 xyz"}'
7
```