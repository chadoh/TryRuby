TRYRUBY!
========
(is now 1.9-ready and hosted using 1.9. Please note this is beta software.)

What is all this?
-----------------
The best way to learn a thing is to [try it][2]!

Try out Ruby. Learn about it. Be guided on a veritable carnival ride of
Ruby wondrousness.

When you come out the other side, you'll likely find that

1. Your computer understands what you're saying! And you don't have much
   trouble speaking its dialect.
2. You have some skill in climbing about on your computer's [command
   line](http://en.wikipedia.org/wiki/Command_line_interface)

It was originally made by a fellow named [_why][1]. But he disappeared. So
some other folks have rebuilt it.

You can help!
=============
It would be wonderful to have some help building this. Here's some
descriptions of what's going on.


What is /irb.cgi?
-----------------
After [_why][1]'s disappearance, the only part of the [TryRuby][2]
implementation that wasn't able to be recovered from archive.org
is the /irb file (or /irb.cgi currently, you can change the name
of the file used in js/irb.js). This script should:

- Take one GET param "cmd", which will contain a single line of
Ruby to run. It can also be "!INIT!IRB", which is called at the
start of the session. The current implementation of irb.cgi
ignores this. "reset" should also do something special (but
doesn't at the moment).
- Return the output, optionally formatted using normal shell
escapes (eg "\033[1;33mThis appears orange")

The output has four main formats for returning a result, error
output, javascript function and line continuation. This is used
by the help system so that it can detect when a user has entered
the next step for the tutorial.

Output results should be formatted with a "=> " at the front of
the output. For example: /irb.cgi?cmd=3*4 should output "=> 12".
Shell escape codes can be used to give the output different colors,
for example "=> \033[1;20m12". This will automatically be removed
when used with the help system. Note that you don't and shouldn't
terminate these shell escapes with \033[m (like with a normal shell).

Standard output should not be prefixed by =>, so /irb.cgi?cmd=puts(44)
should output "44\n=> nil". Here 44 is output, and nil is returned.
Again shell escapes can be used.

Errors should be formated just like standard output, however it should
have no return. For example, /irb.cgi?cmd=non_existant_function should
output something like "\033[1;33mNameError: undefined local variable or
method `non_existant_function' for main:Object".

Javascript functions (such as Popup.goto) should use the format
"\033[1;JsmJAVASCRIPT CODE\033[m". The javascript cod will then be run.
For example, /irb.cgi?cmd=Popup.goto("http://www.google.com") should
output \033[1;JSmwindow.irb.options.popup_goto("http://www.google.com")\033[m.

Finally, if the command isn't finished (eg "def myfunc"), then ".."
should be returned. Eg /irb.cgi?cmd=def%20myfunc should return "..".


How this works with the help system
-----------------------------------
The help system works with the file /tutorials/intro.html. There is a
<div class="stretcher"> section for each part of the tutorial. Most of
it is just the text for that part of the tutorial. However, there is
also a <div class="answer|stdout">regex</div> section. If this matches
against the output for a command, then the tutorial will go to the next
step.

When the class is "answer", then it will match against lines beginning
with `=>`. So `<div class="answer">\d+</div>` will match any code that
returns a number. Other output can be shown as well, eg `someoutput\n=>
42` will match this. The exact regexp is `^\s*=> match_regex_from_help\s*$`.
So the line must be an exact match, other than spaces and the initial `=>`

When the class is "stdout", the match must work on a complete line. Eg
`<div class="stdout">hello</div>` will match `hello`, or `start\n
hello\ngoodbye` (leading spaces are ignored, and multiple lines are
searched. The exact regex is `^\s*match_regex_from_help$`

As a special case, if the tutorial expects that a command isn't finished
(such as with `def myfunc`), then `<div class="stdout">..</div>` will be
used. This is changed from the original implementation [_why][1] used,
which was `<div class="stdout"></div>` as I couldn't get it to work like
that.

Another special case, if the return should some javascript code, then
something like `<div class="stdout">\033\[1;JSm.*popup_goto\(.*\)\033\[m.*</div>`
will be used.

Current Implementation
----------------------
The current implementation doesn't use persistent processes like irb.
What it does is it stores all previous successfully run commands in
`session['previous_commands']`, and runs them all first (disabling `stdout`).
Then it will finally run the command. It uses a primitive method of
detecting incomplete statements (with the function `unfinished_statement?`)
and when a incomplete statement is finished (`finished_statement?`).
The current nesting level (for example,  entering "class X" on one line
and pressing return will have a nesting level of 2) is stored in
`$session['nesting_level']`. All the lines of the current incomplete
statement are stored in `$session['current_statement']`.

To work with javascript functions, a special class `JavascriptResult`
was written, with one accessor, `:js`. If the result of an eval returns
this object, then the output will be formatted in the javascript method.

The sessions are now stored in a tmp folder. For the time being it is
within the htdoc path. I know this is bad. I have included an index.html
that redirects you back to the home page in the mean time.
Ideally, I need to put this some place like /var/logs/tryruby.

From there I will see about the viability of making this use the r bridge so you
can have ruby to R lessons. :-)


  [1]: http://en.wikipedia.org/wiki/Why_the_lucky_stiff
  [2]: http://tryruby.org/
