# Codebraid – live code in Pandoc Markdown

Codebraid is a Python program that enables executable code in
[Pandoc](http://pandoc.org/) Markdown documents.  Using Codebraid can be as
simple as adding a class to your code blocks' attributes, and then running
`codebraid` rather than `pandoc` to convert your document from Markdown to
another format.  `codebraid` supports almost all of `pandoc`'s options and
passes them to `pandoc` internally.

Codebraid currently can run **Python 3.5+**, **Julia**, **Rust**, **R**,
**Bash**, and **JavaScript** code.  Support for additional languages is coming soon.

**Development:**  https://github.com/gpoore/codebraid

View example HTML output, or see the Markdown source or raw HTML (the Python
and Rust examples demonstrate more advanced features at the end):

  * [Python example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/python.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/python.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/python.html)]
  * [Rust example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/rust.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/rust.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/rust.html)]
  * [Julia example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/julia.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/julia.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/julia.html)]
  * [R example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/R.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/R.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/R.html)]
  * [Bash example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/bash.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/bash.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/bash.html)]
  * [JavaScript example](https://htmlpreview.github.com/?https://github.com/gpoore/codebraid/blob/master/examples/javascript.html)
    [[Pandoc Markdown source](https://github.com/gpoore/codebraid/blob/master/examples/javascript.cbmd)]
    [[raw HTML](https://github.com/gpoore/codebraid/blob/master/examples/javascript.html)]


## Simple example

Markdown source `test.md`:

``````markdown
```{.python .cb.run}
var = 'Hello from Python!'
var += ' $2^8 = {}$'.format(2**8)
```

```{.python .cb.run}
print(var)
```
``````

Run `codebraid` (to save the output, add something like `-o test_out.md`, and
add `--overwrite` if it already exists):

```shell
codebraid pandoc --from markdown --to markdown test.md
```

Output:

```markdown
Hello from Python! $2^8 = 256$
```

As this example illustrates, variables persist between code blocks; by
default, code is executed within a single session.  Code output is also cached
by default so that code is only re-executed when modified.


## Features

### Comparison with [Jupyter](https://jupyter.org/), [knitr](https://yihui.name/knitr/), and [Pweave](http://mpastell.com/pweave/)

|                                                | Codebraid | Jupyter Notebook |  knitr   | Pweave   |
|------------------------------------------------|-----------|------------------|----------|----------|
| multiple programming languages per document    | &check;   | &check;&ast;     | &check;† |          |
| multiple independent sessions per language     | &check;   |                  |          |          |
| inline code execution within paragraphs        | &check;   |                  | &check;  | &check;  |
| no out-of-order code execution                 | &check;   |                  | &check;‡ | &check;  |
| no markdown preprocessor or custom syntax      | &check;   | &check;          |          |          |
| minimal diffs for easy version control         | &check;   |                  | &check;  | &check;  |
| hide or display code in final document         | &check;   |                  | &check;  | &check;  |
| insert code output anywhere in a document      | &check;   |                  |          |          |
| can divide code into incomplete snippets       | &check;   |                  | &check;  | &check;  |
| support for literate programming               | &check;   |                  | &check;  |          |
| compatible with any text editor                | &check;   |                  | &check;  | &check;  |

&ast; One primary language per notebook, plus additional languages via
%%script magic.  There is no continuity between %%script cells, because
each cell is executed in a separate process.
<br>
† knitr only provides continuity between code chunks for R, and more recently
Python and Julia.  Code chunks in other languages are executed individually
in separate processes.
<br>
‡ Out-of-order execution is possible with R Markdown notebooks.

<hr>

The table above summarizes Codebraid features in comparison with Jupyter
notebooks, knitr (R Markdown), and Pweave, emphasizing Codebraid's unique
features.  Here are some additional points to consider:

*Jupyter notebooks* — Notebooks have a dedicated, browser-based graphical user
interface.  Jupyter kernels typically allow the code in a cell to be executed
without re-executing any preceding code, providing superior interactivity.
Codebraid has advantages for projects that are more focused on creating a
document than on exploratory programming.

*knitr* — R Markdown documents have a dedicated user interface in R Studio.
knitr provides superior support for R, as well as significant Python and Julia
support that includes R integration.  Codebraid offers continuity between code
chunks for all supported languages, as well as multiple independent sessions
per language.  It also provides unique options for displaying code and its
output.



### More about key features

*Easy debugging* — By default, stderr is shown automatically in the document
whenever there is an error, right next to the code that caused it.  Even
though user code is typically inserted into a template for execution, line
numbers in error messages will correctly correspond with line numbers in code
blocks, because Codebraid tracks the origin of each line of code and
synchronizes error messages.

*Simple language support* — Adding support for a new language can take only
a few minutes.  Codebraid's default system for executing code is based on
writing delimiters to stdout and stderr that allow it to associate code output
with individual code chunks.  Adding a language is as simple as creating a
config file that tells Codebraid which program to run, which file extension to
use, and how to write to stdout and stderr.  See
[`languages/`](https://github.com/gpoore/codebraid/tree/master/codebraid/languages)
for examples.

*No preprocessor* — Unlike many approaches to making code in Markdown
executable, Codebraid is not a preprocessor.  Rather, Codebraid acts on the
abstract syntax tree (AST) that Pandoc generates when parsing a document.
Preprocessors often fail to disable commented-out code blocks because the
preprocessor doesn't recognize Markdown comments.  Preprocessors can also fail
due to the finer points of Markdown parsing.  None of this is an issue for
Codebraid, because Pandoc does the Markdown parsing.

*No custom syntax* — Codebraid introduces no additional Markdown syntax.
Making a code block or inline code executable uses Pandoc's existing syntax
for defining code attributes.



## Installation and requirements

**Installation:**  `pip3 install codebraid` or `pip install codebraid`

Manual installation:  `python3 setup.py install` or `python setup.py install`

**Requirements:**

  * [Pandoc](http://pandoc.org/) 2.4+ (2.7.2+ recommended)
  * Python 3.5+ with `setuptools`, and [`bespon`](https://bespon.org) 0.3
    (`bespon` installation is typically managed by `pip`/`setup.py`)

By default, the `python3` executable will be used to execute code.  If it does
not exist, `python` will be tried to account for Windows and Arch Linux.
Future releases will allow specifying the executable on systems with multiple
Python 3 installations.


## Converting a document

Simply run `codebraid pandoc <normal pandoc options>`.  Note that
`--overwrite` is required for existing files.

`codebraid` should typically be run in the same directory as the document, so
that the default working directory for code is the document directory.  Future
releases will allow customization of the working directory.


## Caching

By default, code output is cached, and code is only re-executed when it is
modified.  The default cache location is a `_codebraid` directory in the
directory with your markdown document.  This can be modified using
`--cache-dir`.  Sharing a single cache location between multiple documents is
not yet supported.

If you are working with external data that changes, you should run `codebraid`
with `--no-cache` to prevent the cache from becoming out of sync with your
data.  Future releases will allow external dependencies to be specified so
that caching will work correctly in these situations.


## Code options

### Commands (Classes)

Code is made executable by adding a Codebraid class to its
[Pandoc attributes](http://pandoc.org/MANUAL.html#fenced-code-blocks).
For example, `` `code`{.python}` `` becomes
`` `code`{.python .cb.run}` ``.

* `.cb.code` — Insert code verbatim, but do not run it.  This is primarily
  useful when combined with other features like naming and then copying code
  chunks.

* `.cb.expr` — Evaluate an expression and interpret the result as Markdown.
  Only works with inline code.

* `.cb.nb` — Execute code in notebook mode.  For inline code, this is
  equivalent to `.cb.expr`.  For code blocks, this inserts the code verbatim,
  followed by any printed output (stdout) verbatim.  If stderr exists, it is
  also inserted verbatim.

* `.cb.paste` — Insert code and/or output copied from one or more named code
  chunks.  The `copy` keyword is used to specify chunks to be copied.  This
  does not execute any code.  If content is copied from multiple code chunks,
  all code chunks must be in the same session and must be in sequential order.
  Unless `show` is specified, display options are inherited from the first
  copied code chunk.

* `.cb.run` — Run code and interpret any printed content (stdout) as Markdown.
  Also insert stderr verbatim if it exists.

### Keyword arguments

Pandoc code attribute syntax allows keyword arguments of the form `key=value`,
with spaces (*not* commas) separating subsequent keys.  `value` can be
unquoted if it contains only letters and some symbols; otherwise, double
quotation marks `"value"` are required.  For example,
```
{.python key1=value1 key2=value2}
```
Codebraid adds support for additional keyword arguments.  In some cases,
multiple keywords can be used for the same option.  This is primarily for
Pandoc compatibility.

#### Execution

* `complete`={`true`, `false`} — By default, code chunks must contain complete
  units of code (function definitions, loops, expressions, and so forth). With
  `complete=false`, this is not required.  Any stdout from code chunks with
  `complete=false` is accumulated until the next code chunk with
  `complete=true` (the default value), or until the end of the session,
  whichever comes first.

  Setting `complete` is incompatible with `outside_main=true`, since the
  `complete` status of code chunks with `outside_main=true` is inferred
  automatically.

* `outside_main`={`true`, `false`} — This allows code chunks to overwrite the
  Codebraid template code.  It is primarily useful for languages like Rust, in
  which code is inserted by default into a `main()` template.  In that case,
  if a session *starts* with one or more code chunks with `outside_main=true`,
  these are used instead of the beginning of the `main()` template.
  Similarly, if a session *ends* with one or more code chunks with
  `outside_main=true`, these are used instead of the end of the `main()`
  template.  If there are any code chunks in between that lack `outside_main`
  (that is, default `outside_main=false`), then these will have their stdout
  collected on a per-chunk basis like normal.  Having code chunks that lack
  `outside_main` is not required; if there are none, the total accumulated
  stdout for a session belongs to the last code chunk in the session.

  `outside_main=true` is incompatible with explicitly setting `complete`.  The
  `complete` status of code chunks with `outside_main=true` is inferred
  automatically.

* `session`={identifier-style string} — By default, all code for a given
  language is executed in a single, shared session so that data and variables
  persist between code chunks.  This allows code to be separated into multiple
  independent sessions.  Session names must be Python-style identifiers.

#### Display

* `first_number`/`startFrom`/`start-from`/`start_from`={integer or `next`} —
  Specify the first line number for code when line numbers are displayed.
  `next` means continue from the last code in the current session.

* `hide`={`markup`, `copied_markup`, `code`, `stdout`, `stderr`, `expr`,
  `all`} — Hide some or all of the elements that are displayed by default.
  Elements can be combined.  For example, `hide=stdout+stderr`.  Note that
  `expr` only applies to `.cb.expr` or `.cb.nb` with inline code, since only
  these evaluate an expression.

* `hide_markup_keys`={key(s)} — Hide the specified code chunk attribute key(s)
  in the Markdown source displayed via `markup` or `copied_markup`.  Multiple
  keys can be specified via `hide_markup_keys=key1+key2`.

  `hide_markup_keys` only applies to the code chunk in which it is used, to
  determined the `markup` for that code chunk.  Thus, it only affects
  `copied_markup` indirectly.

* `line_numbers`/`numberLines`/`number-lines`/`number_lines`={`true`, `false`}
  — Number code lines in code blocks.

* `show`={`markup`, `copied_markup`, `code`, `stdout`, `stderr`, `expr`,
  `none`} — Override the elements that are displayed by default.  `expr` only
  applies to `.cb.expr` and to `.cb.nb` with inline code, since only these
  evaluate an expression.  Elements can be combined.  For example,
  `show=code+stdout`.  Each element displayed can optionally specify a format
  from `raw`, `verbatim`, or `verbatim_or_empty`.  For example,
  `show=code:verbatim+stdout:raw`.

    - `raw` means interpreted as Markdown.
    - `verbatim` produces inline code or a code block, depending on context.
      Nothing is produced if there is no content (for example, nothing in
      stdout.)
    - `verbatim_or_empty` produces inline code containing a single
      non-breaking space or a code block containing a single empty line in the
      event that there is no content.  It is useful when a placeholder is
      desired, or a visual confirmation that there is indeed no output.

  `markup` displays the Markdown source for the inline code or code block.
  Because the Markdown source is not available in the Pandoc AST but rather
  must be recreated from it, the Markdown source displayed with `markup` may
  use a different number of backticks, quote attribute values slightly
  differently, or contain other insignificant differences from the original
  document.

  `copied_markup` displays the Markdown source for code chunks copied via
  `copy`.

  `expr` defaults to `raw` if a format is not specified.  All others default
  to `verbatim`.

#### Copying

* `copy`={chunk name(s)} — Copy one or more named code chunks.  When `copy` is
  used with a command like `.cb.run` that executes code, only the code is
  copied, and it is executed as if it had been entered directly.  When `copy`
  is used with `.cb.code`, only the code is copied and nothing is executed.
  When `copy` is used with `.cb.paste`, both code and output are copied, and
  nothing is executed.  Multiple code chunks may be copied; for example,
  `copy=name1+name2`.  In that case, the code from all chunks is concatenated,
  as is any output that is copied.  Because `copy` brings in code from other
  code chunks, the actual content of a code block or inline code using `copy`
  is discarded.  As a result, this must be empty, or a space or underscore can
  be used as a placeholder.

* `name`={identifier-style string} — Name a code chunk so that it can later be
  copied by name.  Names must be Python-style identifiers.

#### Including external files

* `include_file`={path} — Include the specified file.  A leading `~/` or
  `~<user>/` is expanded to the user's home directory under all operating
  systems, including under Windows with both slashes and backslashes.

  When `include_file` is used with a command like `.cb.run` that executes
  code, the file is included and executed as part of the current session just
  as if the file contents had been entered directly.  When `include_file` is
  used with `.cb.code`, the file is included and displayed just as if it had
  been entered directly.  Because `include_file` brings in code from another
  file, the actual content of a code block or inline code using `include_file`
  is discarded.  As a result, this must be empty, or a space or underscore can
  be used as a placeholder.

* `include_encoding`={encoding} — Encoding for included file.

* `include_lines`={lines/line ranges} — Include the specified lines or line
  ranges.  For example, `1-3,5,7-9,11-`.  Line numbers are one-indexed.  Line
  ranges are inclusive, so `1-3` is `1` up to and including `3`.  If a range
  ends with a hyphen, like `11-`, then everything is included from the line
  through the end of the file.

  Cannot be combined with other `include` options that specify what is to
  be included.

* `include_regex`={regex} — Include the first segment of the file that matches
  the provided regular expression.

  Keep in mind that Pandoc's key-value attributes evaluate backslash escapes
  in values whether or not the values are quoted with double quotation marks,
  so two levels of backslash-escaping are always necessary (one for Pandoc's
  strings, one for the regex itself; there are no raw strings).  Regular
  expressions use *multiline mode*, so `^`/`$` match the start/end of a line,
  and `\A`/`\Z` can be used to match the start/end of the file.  Regular
  expressions use *dotall mode*, so `.` matches anything including the newline
  `\n`; use `[^\n]` when this is not desired.

  Cannot be combined with other `include` options that specify what is to
  be included.

* `include_start_string`={string} — Include everything from the first
  occurrence of this string onward.

  Can only be combined with other `include` options that specify the end of
  what is to be included.

* `include_start_regex`={regex} — Include everything from the first
  match of this regex onward.

  Can only be combined with other `include` options that specify the end of
  what is to be included.  See `include_regex` for notes on regex usage.

* `include_after_string`={string} — Include everything after the first
  occurrence of this string onward.

  Can only be combined with other `include` options that specify the end of
  what is to be included.

* `include_after_regex`={regex} — Include everything after the first
  match of this regex onward.

  Can only be combined with other `include` options that specify the end of
  what is to be included.  See `include_regex` for notes on regex usage.

* `include_before_string`={string} — Include everything before the first
  occurrence of this string.

  Can only be combined with other `include` options that specify the start of
  what is to be included.  If the start is specified, then the first
  occurrence after this point is used, rather than the first occurrence in the
  overall file.

* `include_before_regex`={regex} — Include everything before the first
  match of this regex.

  Can only be combined with other `include` options that specify the start of
  what is to be included.  If the start is specified, then the first match
  after this point is used, rather than the first match in the overall file.
  See `include_regex` for notes on regex usage.

* `include_end_string`={string} — Include everything through the first
  occurrence of this string.

  Can only be combined with other `include` options that specify the start of
  what is to be included.  If the start is specified, then the first
  occurrence after this point is used, rather than the first occurrence in the
  overall file.

* `include_end_regex`={regex} — Include everything through the first
  match of this regex.

  Can only be combined with other `include` options that specify the start of
  what is to be included.  If the start is specified, then the first match
  after this point is used, rather than the first match in the overall file.
  See `include_regex` for notes on regex usage.
