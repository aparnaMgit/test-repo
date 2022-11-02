### Unordered List

+ Create a list by starting a line with `+`, `-`, or `*`
+ Sub-lists are made by indenting 2 spaces:
  - Marker character change forces new list start:
    * Ac tristique libero volutpat at
    + Facilisis in pretium nisl aliquet
    - Nulla volutpat aliquam velit
+ Very easy!
    
### Ordered List

Here is a numbered list:

1. first item
2. second item
3. third item

Note again how the actual text starts at 3 columns in (3 characters
from the left side).

1. You can use sequential numbers...
1. ...or keep all the numbers as `1.`

Start numbering with offset:

57. foo
1. bar

### Nested List

Now a nested list:

1. First, get these ingredients:
   - carrots
   - celery
   - lentils

2. Boil some water.

3. Dump everything in the pot and follow
   this algorithm:
   - find wooden spoon
   - manage pot
      - uncover pot
      - stir
      - cover pot
      - balance wooden spoon precariously on pot handle
   - wait 10 minutes
   - goto first step (or shut off burner when done)

* Do not bump wooden spoon or it will fall.

Notice again how text always lines up on at 3-space indents (including
that last line which continues item 3 above).

### Definition Lists

Apple
   : Pomaceous fruit of plants of the genus Malus in
   the family Rosaceae.

Orange
   : The fruit of an evergreen tree of the genus Citrus.

Tomatoes
   : There's no "e" in tomato.

You can put blank lines in between each of the above definition list lines to spread things
out more. Also you can lazy continue lists, add inline markup, and also include paragraphs
within each definition.

Apple

:   Pomaceous fruit of plants of the genus Malus in
the family Rosaceae (note the lazy continuation).

Orange

:   The fruit of an evergreen tree of the **genus Citrus** (note the inline markup).

Tomatoes

: There's no "e" in tomato.

    { some code, part of Definition 'Tomatoes' }

    Third paragraph of definition 'Tomatoes'.

Here is an example of a _compact definition list style_

Term 1
  ~ Definition 1

Term 2
  ~ Definition 2a
  ~ Definition 2b

## Code

This is an `inline code`.

Here's an indented code block sample:

    # Let me re-iterate ...
    for i in 1 .. 10 { do-something(i) }

As you probably guessed, indented 4 spaces. By the way, instead of
indenting the block, you can use delimited blocks, if you like:

~~~
define foobar() {
    print "Welcome to flavor country!";
}
~~~

(which makes copying & pasting easier). You can optionally mark the
delimited block for syntax highlighting with any code pretty CSS framework.  

PYTHON CODE EXAMPLE:

~~~python
import time
# Quick, count to ten!
for i in range(10):
    # (but not *too* quick)
    time.sleep(0.5)
    print i
~~~

JAVASCRIPT EXAMPLE

``` js
var foo = function (bar) {
  return bar++;
};

console.log(foo(5));
```

PHP CODE EXAMPLE:

~~~php
namespace site\controllers;

use yii\web\Controller;

class BaseController extends Controller {
    const HAPPY = 1;
    const SAD = 2;

    public function actionGood($param) {
        if ($param === self::HAPPY) {
            echo 'I am happy.';
        } else {
            echo 'I am sad.';
        }
    }
}
~~~

## Links
