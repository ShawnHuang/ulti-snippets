This directory contains the main scripts that come bundled with UltiSnips.

Standing On The Shoulders of Giants
===================================

The snippets have been collected from various other project which I want to
express my gratitude for. My main source for inspiration where the following
two projects:

   TextMate: http://svn.textmate.org/trunk/Bundles/
   SnipMate: http://code.google.com/p/snipmate/

All snippets from those sources were copied and cleaned up, so that they are 
  - not using shell script, only python (so they are cross platform compatible)
  - not using any feature that UltiSnips doesn't offer

UltiSnips has seen contributions by various individuals. Those contributions
have been merged into this collection seamlessly and without further comments.

-- vim:ft=rst:nospell:


 Syntax                                                  *UltiSnips-syntax*
=============================================================================

This chapter describes how to write your own snippets and snippet definition
syntax. Examples are used to help illustrate.


4.1 Adding Snippets                               *UltiSnips-adding-snippets*
-------------------

See |UltiSnips-snippet-search-path| for an explanation of where directories
with snippet definitions should be located.

Using a strategy similar to how Vim detects |ftplugins|, UltiSnips iterates
over the snippet definition directories looking for files with names of the
following patterns: ft.snippets, ft_*.snippets, or ft/*, where "ft" is the
'filetype' of the current document and "*" is a shell-like wildcard matching
any string including the empty string. The following table shows some typical
snippet filenames and their associated filetype.

    snippet filename         filetype ~
    ruby.snippets            ruby
    perl.snippets            perl
    c.snippets               c
    c_my.snippets            c
    c/a                      c
    c/b.snippets             c
    all.snippets             *all
    all/a.snippets           *all

* The 'all' filetype is unique. It represents snippets available for use when
editing any document regardless of the filetype. A date insertion snippet, for
example, would fit well in the all.snippets file.

UltiSnips understands Vim's dotted filetype syntax. For example, if you define
a dotted filetype for the CUDA C++ framework, e.g. ":set ft=cuda.cpp", then
UltiSnips will search for and activate snippets for both the cuda and cpp
filetypes.

UltiSnips doesn't understand multiple filetypes in snippet filenames, so a
snippet file named "cuda.cpp.snippets" won't match the "cuda" or "cpp"
filetypes, which means it won't be activated for files with the dotted
filetype "cuda.cpp" either.

The extends directive provides an alternative way of combining snippet files.
When the extends directive is included in a snippet file, it instructs
UltiSnips to include all snippets from the indicated filetypes.

The syntax looks like this: >
   extends ft1, ft2, ft3

For example, the first line in cpp.snippets looks like this: >
   extends c
When UltiSnips activates snippets for a cpp file, it first looks for all c
snippets and activates them as well. This is a convenient way to create
specialized snippet files from more general ones. Multiple "extends" lines are
permitted in a snippet file, and they can be included anywhere in the file.

The snippets file syntax is simple. All lines starting with a # character are
considered comments. Comments are ignored by UltiSnips. Use them to document
snippets.

A line beginning with the keyword 'snippet' marks the beginning of snippet
definition and a line starting with  the keyword 'endsnippet' marks the end.
The snippet definition is placed between the lines. Here is a snippet of an
'if' statement for the Unix shell (sh) filetype.

    snippet if "if ... then (if)"
    if ${2:[[ ${1:condition} ]]}; then
            ${0:#statements}
    fi
    endsnippet

The start line takes the following form: >

   snippet tab_trigger [ "description" [ options ] ]

The tab_trigger is required, but the description and options are optional.

The 'tab_trigger' is the word or string sequence used to trigger the snippet.
Generally a single word is used but the tab_trigger can include spaces. If you
wish to include spaces, you must wrap the tab trigger in quotes. >

    snippet "tab trigger" [ "description" [ options ] ]

The quotes are not part of the trigger. To activate the snippet type: tab trigger
followed by the snippet expand character.

It is not technically necessary to use quotes to wrap a trigger with spaces.
Any matching characters will do. For example, this is a valid snippet starting
line. >
    snippet !tab trigger! [ "description" [ options ] ]

Quotes can be included as part of the trigger by wrapping the trigger in
another character. >
    snippet !"tab trigger"! [ "description" [ options ] ]

To activate this snippet one would type: "tab trigger"

The 'description' is a string describing the trigger. It is helpful for
documenting the snippet and for distinguishing it from other snippets with the
same tab trigger. When a snippet is activated and more than one tab trigger
match, UltiSnips displays a list of the matching snippets with their
descriptions. The user then selects the snippet they want.

The 'options' control the behavior of the snippet. Options are indicated by
single characters. The 'options' characters for a snippet are combined into
a word without spaces.

The options currently supported are: >
   !   Overwrite - A snippet with this option will overwrite all previously
       defined snippets with an identical tab trigger. The default behavior
       is to display list of snippets matching the tab trigger and let the
       user pick the one they want. Use this option to overwrite bundled
       snippets with user defined ones.

   b   Beginning of line - A snippet with this option is expanded only if the
       tab trigger is the first word on the line. In other words, if only
       whitespace precedes the tab trigger, expand. The default is to expand
       snippets at any position regardless of the preceding non-whitespace
       characters.

   i   In-word expansion - By default a snippet is expanded only if the tab
       trigger is the first word on the line or is preceded by one or more
       whitespace characters. A snippet with this option is expanded
       regardless of the preceding character. In other words, the snippet can
       be triggered in the middle of a word.

   w   Word boundary - With this option, the snippet is expanded if
       the tab trigger start matches a word boundary and the tab trigger end
       matches a word boundary. In other words the tab trigger must be
       preceded and followed by non-word characters. Word characters are
       letters, digits and the underscore. Use this option, for example, to
       permit expansion where the tab trigger follows punctuation without
       expanding suffixes of larger words.

   r   Regular expression - With this option, the tab trigger is expected to
       be a python regular expression. The snippet is expanded if the recently
       typed characters match the regular expression. Note: The regular
       expression MUST be quoted (or surrounded with another character) like a
       multi-word tab trigger (see above) whether it has spaces or not. A
       resulting match is passed to any python code blocks in the snippet
       definition as the local variable "match".

   t   Do not expand tabs - If a snippet definition includes leading tab
       characters, by default UltiSnips expands the tab characters honoring
       the Vim 'shiftwidth', 'softtabstop', 'expandtab' and 'tabstop'
       indentation settings. (For example, if 'expandtab' is set, the tab is
       replaced with spaces.) If this option is set, UltiSnips will ignore the
       Vim settings and insert the tab characters as is. This option is useful
       for snippets involved with tab delimited formats, for example.

   s   Remove whitespace immediately before the cursor at the end of a line
       before jumping to the next tabstop.  This is useful if there is a
       tabstop with optional text at the end of a line.

The end line is the 'endsnippet' keyword on a line by itself. >

   endsnippet

When parsing snippet files, UltiSnips chops the trailing newline character
from the 'endsnippet' end line.


 4.1.1 Character Escaping:                     *UltiSnips-character-escaping*

In snippet definitions, the characters '`', '{', '$', '\' and "'" (single
quote) have special meaning. If you want to insert one of these characters
literally, escape them with a backslash, '\'.


4.2 Plaintext Snippets                         *UltiSnips-plaintext-snippets*
----------------------

To illustrate plaintext snippets, let's begin with a simple example. You can
try the examples yourself. Simply edit a new file with Vim. Example snippets
will be added to the 'all.snippets' file, so you'll want to open it in Vim for
editing as well. >
   ~/.vim/UltiSnips/all.snippets

Add this snippet to 'all.snippets' and save the file.

------------------- SNIP -------------------
snippet bye "My mail signature"
Good bye, Sir. Hope to talk to you soon.
- Arthur, King of Britain
endsnippet
------------------- SNAP -------------------

UltiSnips detects when you write changes to a snippets file and automatically
makes the changes active. So in the empty buffer, type the tab trigger 'bye'
and then press the <Tab> key.

bye<Tab> -->
Good bye, Sir. Hope to talk to you soon.
- Arthur, King of Britain

The word 'bye' will be replaced with the text of the snippet definition.


4.3 Visual Placeholder                         *UltiSnips-visual-placeholder*
----------------------

Snippets can contain a special placeholder called ${VISUAL}. The ${VISUAL}
variable is expanded with the text selected just prior to expanding the
snippet.

To see how a snippet with a ${VISUAL} placeholder works, define a snippet with
the placeholder, use Vim's Visual mode to select some text, and then press the
key you use to trigger expanding a snippet (see g:UltiSnipsExpandTrigger). The
selected text is deleted, and you are dropped into Insert mode. Now type the
snippet tab trigger and press the key to trigger expansion. As the snippet
expands, the previously selected text is printed in place of the ${VISUAL}
placeholder.

The ${VISUAL} placeholder can contain default text to use when the snippet has
been triggered when not in Visual mode. The syntax is: >
    ${VISUAL:default text}

The ${VISUAL} placeholder can also define a transformation (see
|UltiSnips-transformations|). The syntax is: >
    ${VISUAL:default/search/replace/option}.

Here is a simple example illustrating a visual transformation. The snippet
will take selected text, replace every instance of "should" within it with
"is" , and wrap the result in tags.

------------------- SNIP -------------------
snippet t
<tag>${VISUAL:inside text/should/is/g}</tag>
endsnippet
------------------- SNAP -------------------

Start with this line of text: >
   this should be cool

Position the cursor on the word "should", then press the key sequence: viw
(visual mode -> select inner word). Then press <Tab>, type "t" and press <Tab>
again. The result is: >
   -> this <tag>is</tag> be cool

If you expand this snippet while not in Visual mode (e.g., in Insert mode type
t<Tab>), you will get: >
   <tag>inside text</tag>


4.4 Interpolation                                   *UltiSnips-interpolation*
-----------------

 4.4.1 Shellcode:                                       *UltiSnips-shellcode*

Snippets can include shellcode. Put a shell command in a snippet and when the
snippet is expanded, the shell command is replaced by the output produced when
the command is executed. The syntax for shellcode is simple: wrap the code in
backticks, '`'. When a snippet is expanded, UltiSnips runs shellcode by first
writing it to a temporary script and then executing the script. The shellcode
is replaced by the standard output. Anything you can run as a script can be
used in shellcode. Include a shebang line, for example, #!/usr/bin/perl, and
your snippet has the ability to run scripts using other programs, perl, for
example.

Here are some examples. This snippet uses a shell command to insert the
current date.

------------------- SNIP -------------------
snippet today
Today is the `date +%d.%m.%y`.
endsnippet
------------------- SNAP -------------------

today<tab> ->
Today is the 15.07.09.


This example inserts the current date using perl.

------------------- SNIP -------------------
snippet today
Today is `#!/usr/bin/perl
@a = localtime(); print $a[3] . '.' . $a[4] . '.' . ($a[5]+1900);`.
endsnippet
------------------- SNAP -------------------
today<tab> ->
Today is 15.6.2009.


 4.4.2 VimScript:                                       *UltiSnips-vimscript*

You can also use Vim scripts (sometimes called VimL) in interpolation. The
syntax is similar to shellcode. Wrap the code in backticks and to distinguish
it as a Vim script, start the code with '!v'. Here is an example that counts
the indent of the current line:

------------------- SNIP -------------------
snippet indent
Indent is: `!v indent(".")`.
endsnippet
------------------- SNAP -------------------
    (note the 4 spaces in front): indent<tab> ->
    (note the 4 spaces in front): Indent is: 4.


 4.4.3 Python:                                             *UltiSnips-python*

Python interpolation is by far the most powerful. The syntax is similar to Vim
scripts except code is started with '!p'. Python scripts can be run using the
python shebang '#!/usr/bin/python', but using the '!p' format comes with some
predefined objects and variables, which can simplify and shorten code. For
example, a 'snip' object instance is implied in python code. Python code using
the '!p' indicator differs in another way. Generally when a snippet is
expanded the standard output of code replaces the code. With python code the
value of the 'rv' property of the 'snip' instance replaces the code. Standard
output is ignored.

The variables automatically defined in python code are: >

   fn   - The current filename
   path - The complete path to the current file
   t    - The values of the placeholders, t[1] is the text of ${1}, and so on
   snip - UltiSnips.TextObjects.SnippetUtil object instance. Has methods that
          simplify indentation handling.

The 'snip' object provides the following methods: >

    snip.mkline(line="", indent=None):
        Returns a line ready to be appended to the result. If indent
        is None, then mkline prepends spaces and/or tabs appropriate to the
        current 'tabstop' and 'expandtab' variables.

    snip.shift(amount=1):
        Shifts the default indentation level used by mkline right by the
        number of spaces defined by 'shiftwidth', 'amount' times.

    snip.unshift(amount=1):
        Shifts the default indentation level used by mkline left by the
        number of spaces defined by 'shiftwidth', 'amount' times.

    snip.reset_indent():
        Resets the indentation level to its initial value.

    snip.opt(var, default):
        Checks if the Vim variable 'var' has been set. If so, it returns the
        variable's value; otherwise, it returns the value of 'default'.

The 'snip' object provides some properties as well: >

    snip.rv:
        'rv' is the return value, the text that will replace the python block
        in the snippet definition. It is initialized to the empty string. This
        deprecates the 'res' variable.

    snip.c:
        The text currently in the python block's position within the snippet.
        It is set to empty string as soon as interpolation is completed. Thus
        you can check if snip.c is != "" to make sure that the interpolation
        is only done once. This deprecates the "cur" variable.

    snip.v:
         Data related to the ${VISUAL} placeholder. The property has two
         attributes:
             snip.v.mode   ('v', 'V', '^V', see |visual-mode| )
             snip.v.text   The text that was selected.

    snip.fn:
        The current filename.

    snip.basename:
        The current filename with the extension removed.

    snip.ft:
        The current filetype.

For your convenience, the 'snip' object also provides the following
operators: >

    snip >> amount:
        Equivalent to snip.shift(amount)
    snip << amount:
        Equivalent to snip.unshift(amount)
    snip += line:
        Equivalent to "snip.rv += '\n' + snip.mkline(line)"

Any variables defined in a python block can be used in other python blocks
that follow within the same snippet. Also, the python modules 'vim', 're',
'os', 'string' and 'random' are pre-imported within the scope of snippet code.
Other modules can be imported using the python 'import' command.

Python code allows for very flexible snippets. For example, the following
snippet mirrors the first tabstop value on the same line but right aligned and
in uppercase.

------------------- SNIP -------------------
snippet wow
${1:Text}`!p snip.rv = (75-2*len(t[1]))*' '+t[1].upper()`
endsnippet
------------------- SNAP -------------------
wow<tab>Hello World ->
Hello World                                                     HELLO WORLD

The following snippet uses the regular expression option and illustrates
regular expression grouping using python's match object. It shows that the
expansion of a snippet can depend on the tab trigger used to define the
snippet, and that tab trigger itself can vary.

------------------- SNIP -------------------
snippet "be(gin)?( (\S+))?" "begin{} / end{}" br
\begin{${1:`!p
snip.rv = match.group(3) if match.group(2) is not None else "something"`}}
    ${2:${VISUAL}}
\end{$1}$0
endsnippet
------------------- SNAP -------------------
be<tab>center<c-j> ->
\begin{center}

\end{center}
------------------- SNAP -------------------
be center<tab> ->
\begin{center}

\end{center}

The second form is a variation of the first; both produce the same result,
but it illustrates how regular expression grouping works. Using regular
expressions in this manner has some drawbacks:
1. If you use the <Tab> key for both expanding snippets and completion then
   if you typed "be form<Tab>" expecting the completion "be formatted", you
   would end up with the above SNAP instead, not what you want.
2. The snippet is harder to read.


 4.4.4 Global Snippets:                                   *UltiSnips-globals*

Global snippets provide a way to reuse common code in multiple snippets.
Currently, only python code is supported. The result of executing the contents
of a global snippet is put into the globals of each python block in the
snippet file. To create a global snippet, use the keyword 'global' in place of
'snippet', and for python code, you use '!p' for the trigger. For example, the
following snippet produces the same output as the last example . However, with
this syntax the 'upper_right' snippet can be reused by other snippets.

------------------- SNIP -------------------
global !p
def upper_right(inp):
    return (75 - 2 * len(inp))*' ' + inp.upper()
endglobal

snippet wow
${1:Text}`!p snip.rv = upper_right(t[1])`
endsnippet
------------------- SNAP -------------------
wow<tab>Hello World ->
Hello World                                                     HELLO WORLD

Python global functions can be stored in a python module and then imported.
This makes global functions easily accessible to all snippet files.

First, add the directory modules are stored in to the python search path. For
example, add this line to your vimrc file. >

   py import sys; sys.path.append("/home/sirver/.vim/python")

Now, import modules from this directory using a global snippet in your snippet
file >

   global !p
   from my_snippet_helpers import *
   endglobals


4.4 Tabstops and Placeholders   *UltiSnips-tabstops* *UltiSnips-placeholders*
-----------------------------

Snippets are used to quickly insert reused text into a document. Often the
text has a fixed structure with variable components. Tabstops are used to
simplify modifying the variable content. With tabstops you can easily place
the cursor at the point of the variable content, enter the content you want,
then jump to the next variable component, enter that content, and continue
until all the variable components are complete.

The syntax for a tabstop is the dollar sign followed by a number, for example,
'$1'. Tabstops start at number 1 and are followed in sequential order. The
'$0' tabstop is a special tabstop. It is always the last tabstop in the
snippet no matter how many tabstops are defined.

Here is a simple example.

------------------- SNIP -------------------
snippet letter
Dear $1,
$0
Yours sincerely,
$2
endsnippet
------------------- SNAP -------------------
letter<tab>Ben<c-j>Paul<c-j>Thanks for suggesting UltiSnips!->
Dear Ben,
Thanks for suggesting UltiSnips!
Yours sincerely,
Paul


You can use <c-j> to jump to the next tabstop, and <c-k> to jump to the
previous. The <Tab> key was not used for jumping forward because many people
(myself included) use <Tab> for completion. See |UltiSnips-triggers| for
help on defining different keys for tabstops.

Many times it is useful to have some default text for a tabstop. The default
text may be a value commonly used for the variable component, or it may be a
word or phrase that reminds you what is expected for the variable component.
To include default text, the syntax is '${1:value}'.

The following example illustrates a snippet for the shell 'case' statement.
The tabstops use default values to remind the user of what value is expected.

------------------- SNIP -------------------
snippet case
case ${1:word} in
    ${2:pattern} ) $0;;
esac
endsnippet
------------------- SNAP -------------------

case<tab>$option<c-j>-v<c-j>verbose=true
case $option in
    -v ) verbose=true;;
esac


Sometimes it is useful to have a tabstop within a tabstop. To do this, simply
include the nested tabstop as part of the default text. Consider the following
example illustrating an HTML anchor snippet.


------------------- SNIP -------------------
snippet a
<a href="${1:http://www.${2:example.com}}"</a>
    $0
</a>
endsnippet
------------------- SNAP -------------------

When this snippet is expanded, the first tabstop has a default value of
'http://www.example.com'. If you want the 'http://' schema, jump to the next
tabstop. It has a default value of 'example.com'. This can be replaced by
typing whatever domain you want.

a<tab><c-j>google.com<c-j>Google ->
<a href="http://www.google.com">
    Google
</a>

If at the first tabstop you want a different url schema or want to replace the
default url with a named anchor, '#name', for example, just type the value you
want.

a<tab>#top<c-j>Top ->
<a href="#top">
    Top
</a>

In the last example, typing any text at the first tabstop replaces the default
value, including the second tabstop, with the typed text. So the second
tabstop is essentially deleted. When a tabstop jump is triggered, UltiSnips
moves to the next remaining tabstop '$0'. This feature can be used
intentionally as a handy way for providing optional tabstop values to the
user. Here is an example to illustrate.

------------------- SNIP -------------------
snippet a
<a href="$1"${2: class="${3:link}"}>
    $0
</a>
endsnippet
------------------- SNAP -------------------

Here, '$1' marks the first tabstop. It is assumed you always want to add a
value for the 'href' attribute. After entering the url and pressing <c-j>, the
snippet will jump to the second tabstop, '$2'. This tabstop is optional. The
default text is ' class="link"'. You can press <c-j> to accept the tabstop,
and the snippet will jump to the third tabstop, '$3', and you can enter the
class attribute value, or, at the second tabstop you can press the backspace
key thereby replacing the second tabstop default with an empty string,
essentially removing it. In either case, continue by pressing <c-j> and the
snippet will jump to the final tabstop inside the anchor.

a<tab>http://www.google.com<c-j><c-j>visited<c-j>Google ->
<a href="http://www.google.com" class="visited">
    Google
</a>

a<tab>http://www.google.com<c-j><BS><c-j>Google ->
<a href="http://www.google.com">
    Google
</a>


The default text of tabstops can also contain mirrors, transformations or
interpolation.


4.6 Mirrors                                               *UltiSnips-mirrors*
-----------

Mirrors repeat the content of a tabstop. During snippet expansion when you
enter the value for a tabstop, all mirrors of that tabstop are replaced with
the same value. To mirror a tabstop simply insert the tabstop again using the
"dollar sign followed by a number" syntax, e.g., '$1'.

A tabstop can be mirrored multiple times in one snippet, and more than one
tabstop can be mirrored in the same snippet. A mirrored tabstop can have a
default value defined. Only the first instance of the tabstop need have a
default value. Mirrored tabstop will take on the default value automatically.

Mirrors are handy for start-end tags, for example, TeX 'begin' and 'end' tag
labels, XML and HTML tags, and C code #ifndef blocks. Here are some snippet
examples.

------------------- SNIP -------------------
snippet env
\begin{${1:enumerate}}
    $0
\end{$1}
endsnippet
------------------- SNAP -------------------
env<tab>itemize ->
\begin{itemize}

\end{itemize}

------------------- SNIP -------------------
snippet ifndef
#ifndef ${1:SOME_DEFINE}
#define $1
$0
#endif /* $1 */
endsnippet
------------------- SNAP -------------------
ifndef<tab>WIN32 ->
#ifndef WIN32
#define WIN32

#endif /* WIN32 */


4.7 Transformations                               *UltiSnips-transformations*
-------------------

Note: Transformations are a bit difficult to grasp so this chapter is divided
into two sections. The first describes transformations and their syntax, and
the second illustrates transformations with demos.

Transformations are like mirrors but instead of just copying text from the
original tabstop verbatim, a regular expression is matched to the content of
the referenced tabstop and a transformation is then applied to the matched
pattern. The syntax and functionality of transformations in UltiSnips follow
very closely to TextMate transformations.

A transformation has the following syntax: >
   ${<tab_stop_no/regular_expression/replacement/options}

The components are defined as follows: >
   tab_stop_no        - The number of the tabstop to reference
   regular_expression - The regular expression the value of the referenced
                        tabstop is matched on
   replacement        - The replacement string, explained in detail below
   options            - Options for the regular expression

The options can be any combination of >
   g    - global replace
          By default, only the first match of the regular expression is
          replaced. With this option all matches are replaced.
   i    - case insensitive
          By default, regular expression matching is case sensitive. With this
          option, matching is done without regard to case.

The syntax of regular expressions is beyond the scope of this document. Python
regular expressions are used internally, so the python 're' module can be used
as a guide. See http://docs.python.org/library/re.html.

The syntax for the replacement string is unique. The next paragraph describes
it in detail.


 4.7.1 Replacement String:                     *UltiSnips-replacement-string*

The replacement string can contain $no variables, e.g., $1, which reference
matched groups in the regular expression. The $0 variable is special and
yields the whole match. The replacement string can also contain special escape
sequences: >
   \u   - Uppercase next letter
   \l   - Lowercase next letter
   \U   - Uppercase everything till the next \E
   \L   - Lowercase everything till the next \E
   \E   - End upper or lowercase started with \L or \U
   \n   - A newline
   \t   - A literal tab

Finally, the replacement string can contain conditional replacements using the
syntax (?no:text:other text). This reads as follows: if the group $no has
matched, insert "text", otherwise insert "other text". "other text" is
optional and if not provided defaults to the empty string, "". This feature
is very powerful. It allows you to add optional text into snippets.


 4.7.2 Demos:                                               *UltiSnips-demos*

Transformations are very powerful but often the syntax is convoluted.
Hopefully the demos below help illustrate transformation features.

Demo: Uppercase one character
------------------- SNIP -------------------
snippet title "Title transformation"
${1:a text}
${1/\w+\s*/\u$0/}
endsnippet
------------------- SNAP -------------------
title<tab>big small ->
big small
Big small


Demo: Uppercase one character and global replace
------------------- SNIP -------------------
snippet title "Titlelize in the Transformation"
${1:a text}
${1/\w+\s*/\u$0/g}
endsnippet
------------------- SNAP -------------------
title<tab>this is a title ->
this is a title
This Is A Title


Demo: Regular expression grouping
      This is a clever c-like printf snippet, the second tabstop is only shown
      when there is a format (%) character in the first tabstop.

------------------- SNIP -------------------
snippet printf
printf("${1:%s}\n"${1/([^%]|%%)*(%.)?.*/(?2:, :\);)/}$2${1/([^%]|%%)*(%.)?.*/(?2:\);)/}
endsnippet
------------------- SNAP -------------------
printf<tab>Hello<c-j> // End of line ->
printf("Hello\n"); // End of line

But
printf<tab>A is: %s<c-j>A<c-j> // End of line ->
printf("A is: %s\n", A); // End of line


There are many more examples of what can be done with transformations in the
bundled snippets.


4.8 Clearing snippets                           *UltiSnips-clearing-snippets*

To remove snippets for the current file type, use the 'clearsnippets'
directive.

------------------- SNIP -------------------
clearsnippets
------------------- SNAP -------------------

Without arguments, 'clearsnippets' removes all snippets defined up to that
point far for the current file type.  Just a reminder, by default UltiSnips
traverses 'runtimepath' in reverse order, so 'clearsnippets' removes snippet
definitions appearing in files in 'runtimepath' after the '.snippets' file in
which it is encountered.

To clear one or more specific snippet, provide the names of the snippets as
arguments to the 'clearsnippets' command. The following example will clear
the snippets 'trigger1' and 'trigger2'.


------------------- SNIP -------------------
clearsnippets trigger1 trigger2
------------------- SNAP -------------------

UltiSnips comes with a set of default snippets for many file types. If you
would rather not have some of them defined, you can use 'clearsnippets' in
your personal snippets files to remove them. Note: you do not need to remove
snippets if you wish to replace them. Simply use the '!' option. (See
|UltiSnips-adding-snippets| for the options).



