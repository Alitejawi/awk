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