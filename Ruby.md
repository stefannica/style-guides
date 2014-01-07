# Ruby style guide

## Background

These are coding conventions used by all SUSE projects written in Ruby (such as
[SUSE Cloud](https://www.suse.com/products/suse-cloud/), [SUSE Studio](http://susestudio.com), [WebYaST]
(http://webyast.github.com/webyast/), [OBS](http://build.opensuse.org) and
[SLMS](https://www.suse.com/products/susestudio/features/lifecycle-management.html)).
We are making them public for the benefit of our external contributors and the
Ruby community in general. The original author and maintainer is [David
Majda](https://github.com/dmajda).

## General rules

We aim to write code that is readable, understandable and easily modifiable.
All other considerations (such as performance) are secondary and should apply
only in warranted cases. We especially try to avoid unnecessary cleverness.

When adding new code, use conventions of the code around the change, even if it
does not conform to the conventions described here. Local consistency is more
important than global one. Never change the code to conform the conventions and
change the functionality in one commit — split such work into two or more
separate changes. Also consider that by formatting changes you are disrupting
useful commands like `git blame`.

For every rule there is a situation when it makes sense to break it. Consider
rules in this document as strong recommendations, follow them as much as
possible, but if you need to break them, do it. Just make sure you have a good
reason and you don’t do that too often.

## Code layout

1. **Wrap lines at 80 columns.** This is not a hard rule, but if you break it,
	 it should be only because the code would be less readable otherwise (e.g.
   you are embedding a long URL).

   Rationale: While long lines are not a problem on wide-screen displays, many
   people look at the code on terminals, e-mail clients, etc. where the viewport
   isn’t that wide. People also use multiple editor windows side-by-side (e.g.
   when writing code and tests for it simultaneously)
   or viewing diffs.

1. **Use two spaces for indentation. Tabs are forbidden.**

1. **Avoid trailing whitespace.**

   Tip: You can highlight trailing whitespace in vim by adding the following
   lines in your `~/.vimrc`:

   ```vim
   " Highlight traling whitespace
   autocmd ColorScheme * highlight TrailingWhitespace ctermbg=red guibg=red
   autocmd InsertEnter * match TrailingWhitespace /some nonsense/
   autocmd InsertLeave * match TrailingWhitespace /\s\+$/}}
   ```

## Comments

1. **Begin all comments with `#` followed by a space. Do not use
   `=begin`/`=end` multi-line comments.**

   Rationale: The space helps readability. The `=begin`/`=end` comments often
   confuse syntax highlighters and various tools, and decreases scanability of
   the code (it’s less obvious that something is a comment from the first
   look).

1. **Comments should not duplicate the code, but explain things that are not
   possible to express in it.** These are mainly:
    - High-level overviews of functionality (not expressible in code because it
      is too detailed)
    - Reasoning behind the code
    - Warning for hacks/ugliness where it is necessary for some reason (though
      you should try to avoid hacks/ugliness in the first place)

   You should especially avoid “header comments” dividing sections of code —
   these are usually better handled by separating the code into well-named
   methods. For example, instead of writing sequential code like this:

   ```ruby
   # Step 1: Import repositories

   # …20 lines of repository importing code…

   # Step 2: Import packages

   # …20 lines of package importing code…
   ```

   Refactor the separate parts into methods:

   ```ruby
   def import_repositories
      # …20 lines of repository importing code…
   end

   def import_packages
      # …20 lines of package importing code…
   end

   import_repositories
   import_packages
   ```

1. **Prefer stable references instead of verbose inline descriptions.**
   Sometimes instead of describing something directly it is better to reference
   an external source (Bugzilla, blog post, etc). This usually saves space and
   some duplication but requires reader to look up the information.  The
   external source can also disappear or be inaccessible for the reader. Use
   your common sense to decide whether reference or direct description is
   better on case by case basis.

1. **All classes should have a comment describing what it represents.** Methods
   can have a describing comment too, but it is not strictly necessary unless
   documentation is automatically generated from the comments.

1. **Almost all comments should be sentences starting with a capital letter,
   and ending with a period.**

1. **Avoid commented-out code in the source.** Disabled code should be just
   deleted (it will still be stored by the VCS) or put behind a configuration
   flag (the exact mechanism is project-dependent).

   Rationale: Commented-out code has a tendency to stay in the code forever.
   After a while nobody can remember why it was commented out and decide if it
   is actually safe to delete it now. So it will stay there, slowly bitrotting,
   and just creating a noise and distraction.

## Literals

### Strings

1. **Write all strings surrounded with double quotes.** Single quotes should be
   used only in cases where interpolation is really not desired (e.g. you have
   a string containing the `#` character).

   Rationale: In many languages, single quotes are used for characters, not
   strings. This rule makes it easier to work in multi-language environment.

1. **Prefer string interpolation to concatenating using the `+` operator.**
   However, use string interpolation with short expressions only. If the
   interpolated expression is too long, save its value into a variable in front
   of the string literal and use that variable inside the string.

1. **For multi-line strings, you can use either the heredoc syntax or create
   the string using concatenation.** Prefer heredoc syntax for longer strings
   and for cases where you don't care about whitespace inside the string. You
   can then use the version of syntax that does not disrupt indentation:

   ```ruby
   html = <<-EOT
     <html>
       <head>
         <title></title>
       </head>
       <body>
       </body>
     </html>
   EOT
   ```

1. **Prefer concatentation for cases where whitespace would be a problem or for
   shorter strings**:

   ```ruby
   script = "#!/bin/sh\n" +
     "rm -rf /\n" +
     "echo 'Surprised?'"
   ```

   Avoid the `%`, `%q` and `%Q` alternative syntaxes for strings.

   Rationale: Little less escape characters do not outweigh the loss of syntax
   regularity (which is useful when munging source code with regular
   expressions/scripts) and confusion of non-Ruby programmers reading Ruby
   code.

### Arrays

1. Write short arrays on one line like this:

   ```ruby
   [1, 2, 3]
   ```

1. Write longer arrays on multiple lines like this:

   ```ruby
   [
     1,
     2,
     3
   ]
   ```

   Rationale: Placing the square brackets on separate lines and indenting the
   elements makes beginning and end of the array literal more visible. This is
   especially useful when arrays are nested. Ruby interpreter permits trailing
   comma after the least item, but this should not be used as it is not logical
   (comma is a separator), it makes the last item less recognizable and it is
   inconsistent with other languages.

1. Avoid the `%w` alternative syntax for arrays.

   Rationale: Little less quotes do not outweigh the loss of syntax regularity
   (which is useful when munging source code with regular expressions/scripts)
   and confusion of non-Ruby programmers reading Ruby code.

### Hashes

1. Write short hashes on one line like this:

   ```ruby
   { :a => 1, :b => 2, :c => 3 }
   ```

1. Write longer hashes on multiple lines like this:

   ```ruby
   {
     :a   => 1,
     :foo => 2,
     :c   => {
       :d   => 4,
       :bar => 5
     }
   }
   ```

   Rationale: Alignment of the arrows helps readability. Nested hashes are
   indented only by additional two spaces because otherwise the nested hash
   would be too far on the right. Ruby interpreter permits trailing comma after
   the least item, but this should not be used as it is not logical (comma is a
   separator), it makes the last item less recognizable and it is inconsistent
   with other languages.

### Regular expressions

Writing good regular expressions is tricky. Always consider what inputs the
expression may encounter and how flexible the regular expression should be. If
you know in advance what values will get fed in, you should write the
expression as strictly as possible, so that an unexpected value will trigger an
error. On the other hand, if the expression will be used in unknown environment
or on wide variety of inputs, it usually makes sense to make it more
permissive.

1. If the regular expression is complex or very long, use the extended form
   (enabled using the `/x` flag) with indentation, whitespace and comments to
   indicate structure and meaning. See e.g. “Define the messy regexen up here”
   section of http://www.jwz.org/hacks/mork.pl for good examples of this in
   Perl.

1. Avoid the `%r` alternative syntax for regular expressions.

   Rationale: Little less escape characters do not outweigh the loss of syntax
   regularity (which is useful when munging source code with regular
   expressions/scripts) and confusion of non-Ruby programmers reading Ruby
   code.

## Expressions

### Variables

Strive to make variable names short and meaningful. Generally longer variable
scope means a need for longer (more precisely qualified) name. Use
`lower_case_underscore_separated_words` for variable names.

### Operators

1. Always put at least one space around all operators:

   ```ruby
   1 + 2 * 3
   ```

1. When multiple symmetrical expressions are on lines one after another, align
   their operators. On the other hand, do not align operators of unrelated
   expression which look similar just by coincidence.

   Rationale: Alignment helps reader recognize that the expressions are related
   or symmetrical.

   When splitting an expression into multiple lines, write the operator around
   which you split the expression at the end of the line (this is actually
   enforced by Ruby syntax) and indent the rest of the expression by two
   spaces:

   ```ruby
   very_long_expression_foo +
     very_long_expression_bar +
     very_long_expression_baz
   ```

### Assignment

1. Always put one space around the assignment operator:

   ```ruby
   a = 5
   ```

1. When multiple symmetrical assignments are on lines one after another, align
   their assignment operators. On the other hand, do not align assignment
   operators of unrelated assignments which look similar just by coincidence.

   Rationale: Alignment helps reader recognize that the assignments are related
   or symmetrical.

1. Avoid multiple assignment except in two cases:

    1. When a method returns an array of items which you want to assign into
       separate variables:

       ```ruby
       x, y = polar_to_cartesian(r, phi)
       ```

    1. Inside a constructor where you are assigning passed parameters into
       instance variables:

       ```ruby
       def initialize(x, y)
          @x, @y = x, y
       end
       ```

    1. Use idiomatic assignment operator `||=` when appropriate. Typical use is
       to initialize some variable lazily (usually to save memory or time):

       ```ruby
       @plugins ||= load_plugins
       ```

## Statements

### `if` and `unless` statements

1. Generally, use multi-line `if` statements:

   ```ruby
   if foo
      bar
   elsif baz
      qux
   else
      quux
   end
   ```

1. If there are no `elsif` or `else` clauses and both the condition and the
   statement in the `if` clause are short, use the trailing form:

   ```ruby
   foo if bar
   ```

   This form is especially useful for handling corner cases at the beginning of
   methods:

   ```ruby
   def print_user(user)
      return if user.invalid?
      puts user.name
   end
   ```

   Never nest trailing form or combine it with other trailing statements.

1. If the condition of an `if` statement is too long to fit on one line, do not
   wrap it. Instead, extract the value into one or more variables in front of
   the `if` statement and use the variable(s) in the condition.

   Rationale: If a condition does not fit into one line, it usually means it is
   too complex. Extraction into a variable and naming that variable will
   hopefully encourage you to think about the condition and possible
   simplifications. Even if you won’t find any simplification, the variable
   name will help readers to better understand the condition.

1. Do not use the unless statement in its multi-line form, only in its trailing
   form.

   Rationale: When reading an unless statement, one must usually mentally
   negate the condition to understand the statement. For a trailing form, this
   is fairly natural, as the statement resembles English sentence, but for the
   multi-line form it is not as easy.  When testing objects for nil value, use
   the implicit rule that nil is equivalent to false in boolean context. For
   example, write:

   ```ruby
   puts user.name if user            # Use this form
   ```

   instead of

   ```ruby
   puts user.name if !user.nil?      # Not this
   ```

   or

   ```ruby
   puts user.name unless user.nil?   # And not this
   ```

   Rationale: While conditions should generally be boolean values, proper
   variable naming usually ensures that a variable containing a generic object
   is easily distinguishable form a one containing boolean and there is usually
   no confusion about semantics. This is aided by the fact that Ruby has no
   falsey values except `false` and `nil`. The result is shorter code.

### `while` and `until` statements

1. Generally, use multi-line `while`/`until` statements:

   ```ruby
   while foo > 0
      bar
   end
   ```

   Do not use the trailing form.

   Rationale: Trailing form of `while`/`until` statements is poorly designed.
   If you use just one statement as a body and the condition is falsey, the
   body is not run at all. However if you use `begin`…`end` as a body, it is
   run at least once before the condition is even evaluated. Hmm, or was it the
   other way? If I don’t remember, neither will any less experienced Ruby
   programmer.

   Rules for the condition of `while`/`until` statements are the same as for
   `if`/`unless`.


### Method calls

#### Parameters

When calling a method, generally use parentheses around passed parameters. Avoid them only in these cases:

1. When using a DSL where parentheses usually are not used. Often these are
   method calls which take a block and mimic a regular language statements —
   e.g. `task` in Rake tasks or `describe` and `it` in RSpec tests. Class
   “attributes” like ActiveRecord validations or associations are another
   common form.

1. When calling a statement-like method. Statement-like methods are usually
   called without explicit receiver and in such a way that they do not return
   any value. Often they have only one or two parameters. Typical examples are:

   - printing methods like `puts`
   - logging methods
   - assertion methods
   - system-calling methods (`system`, `exec`, …)

1. When you find the code significantly more readable without the parentheses.

1. When calling a method without any parameters.

When the parameters do not fit on one line, write the method call like this:

```ruby
foo(bar, baz,
  qux, quux)
```

That is, break the parameter list on appropriate places, indent the
continuation and do not necessarily put each parameter on separate line.

For DSLs calls with omitted parentheses, write the method call like this:

```ruby
foo bar,
    baz,
    qux,
    quux
```

If the method takes a hash as a last parameter, do not enclose it in `{`…`}`,
just inline the keys and values among regular parameters (and align the arrows
if each parameter is on separate line). If it takes more than one hash as
parameters, enclose all of them (including the last one) in `{`…`}` and treat
them each as a single parameter.

### Blocks

1. For one-line blocks, use the `{`…`}` syntax. For multi-line blocks, use the
   `do`…`end` syntax.

1. Do not call methods on results of multi-line blocks. Save the result into a
   variable and call the methods on it instead. For example, instead of:

   ```ruby
   lines = people.map do |person|
      person.name + " " + person.surname
   end.join("\n")
   ```

   write:

   ```ruby
   names = people.map do |person|
      person.name + " " + person.surname
   end
   lines = names.join("\n")
   ```

#### Parameters

Rules for block parameters are basically the same as for method parameters.
However, some special rules for parameter names apply:

1. When iterating a collection (using `each`, `map`, etc.), the block parameter
   should be singular name of the collection member:

   ```ruby
   users.each do |user|
      puts "#{user.name}"
   end
   ```

1. For one-line block bodies, the block parameter name should be just the first
   letter of the name of the collection member:

   ```ruby
   users.each { |u| puts "#{u.name}" }
   ```

1. For some common methods, the name should be the conventional one:

   ```ruby
   File.open { |f| … }
   ```

### Exceptions

Exceptions are a nice mechanism that allows one to report errors with all
associated information and process them on appropriate place without cluttering
the code with return value checks. This power is easily misused.

#### Raising exceptions

1. Raise exceptions only in exceptional situations. Do not abuse them as
   control-flow mechanism (“long-distance goto”).

1. Always try to raise exception of appropriate type and on the correct level of
   abstraction. Do not allow low-level exceptions (e.g. from network, database,
   or filesystem) leak through your interface.

   Rationale: Leaking low-level exceptions leaks information about the
   implementation to the outside. Someone may start to depend on this
   information and problems may arise when the implementation needs to change.

1. Get familiar with standard Ruby exception classes and try to use them as
   much as possible. But do not limit yourself to them and introduce new
  exception classes as needed.

#### Handling exceptions

1. Do not use generic rescue exception handler (`rescue` or `rescue Exception
   => e`) and rescue tail form (`foo = bar rescue nil`). Notable exception from
   this rule are top-level catch-all handlers in scripts or various components
   that generally just log the exception and/or present some error message to
   users.

   Rationale: By catching only exceptions you anticipate you make sure
   unanticipated exceptions will pass through your handler and will be handled
   by a generic top-level exception handler (either the Ruby one or a custom
   one). This usually makes them visible. Since unanticipated exceptions often
   mean bugs in the code, this strategy ensures that bugs won’t go unnoticed.
   The opposite (using catch-all handlers) would hide them.

1. Name the variable which stores the exception in the exception handler `e`.
   Do not use the `$!` variable to access the last thrown exception.

   Rationale: Using `$!` is fragile. It easily breaks when exception is raised
   in an exception handler or code is moved around.

1. When using the rescue statement to wrap the whole method body, do not use
   separate `begin`…`end` around it, but reuse `def`….`end` of the method:

   ```ruby
   def foo
      bar
   rescue BazError => e
      qux
   end
   ```

#### Defining exception classes

1. Make your exception classes subclasses of `StandardError` so they are
   caught by generic rescue statements.

1. Define your exception classes inside the class or module that raises them
   (as much “inside” as possible). Do not define top-level exception classes
   unless they are project-wide classes intended to be used everywhere.

## Definitions

### Method definitions

Write short methods that do one thing well and nothing more. Once you need to
scroll to view method’s beginning/end, you have sections with comment headings
or you have more than 3 levels of indentation, your method is probably too
complex and should be split.

### Names

1. Use short and descriptive method names. Above all, they should make sense at
   the place where the method is called. Use
   `lower_case_underscore_separated_words` for method names.

1. Except for accessors and query methods, method names should generally
   contain a verb indicating the action of the method. Try to avoid generic
   names such as `process`, `handle`, `do`, etc.

1. Method names should almost never use conjugations like “and”, “or”, etc. It
   usually means that the method is doing two or more things and that it should
   be split up.

1. Use the `?` suffix for methods returning boolean value. Do not use the `is_`
   prefix for them at the same time, it’s redundant.

1. Sometimes you can have a problem with coming up with suitable name for a
   method. In that case, consider the possibility that it is just a symptom of
   a bigger problem, like unclear responsibility of the method or trying to do
   too much.

### Parameters

1. Always use parentheses around method parameters, except when the method does
   not have any.

   Rationale: Parentheses help to separate the method name form the parameters
   and thus increase readability, especially when just quickly scanning the
   code. Parameter separated from a method name only by a whitespace can be
   easily mistaken for a part of the method name.

1. Method parameters names should be short and descriptive. They should make
   sense to a person looking at the method signature without any documentation.
   Use `lower_case_underscore_separated_words` for parameter names.

   Good example:

   ```ruby
   def copy_log(source_file, dest_file)
   ```

   Bad example:

   ```ruby
   def copy_log(file1, file2)
   ```

   It is often a good idea to use such a method and parameter names so that the
   result looks like natural English sentence — especially at the call site.
   There are many good examples in the Ruby standard library and Rails.

1. If a method defines more than one optional parameter, seriously consider
   simulating keyword parameters using a hash parameter.

   Rationale: This will usually yield better interface for the caller since he
   will see parameter names at the call site. This is especially true for
   boolean parameters. See
   [FileUtils#copy_entry](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/fileutils/rdoc/FileUtils.html#M000905)
   for a negative example.

1. If a method takes a hash parameter as the last one, it should be named
   `options`.

1. If a method takes a splat parameter (beginning with `*`), it should be named
   by the collection it represents. If it is too generic (as is often the case
   in `method_missing`), it should be named `args`.

1. If a method takes a block parameter (beginning with `&`), it should be named
   `block`.

### Class definitions

1. Write short classes that do one thing well and nothing more. Think about how
   the class will be used by its users and design the API accordingly.

1. Split class definition into the following sections:
    - Constant definitions
    - Various definitions (e.g. ActiveRecord validations and associations)
    - Initialization code
    - Singleton methods
    - Public methods
    - Protected methods
    - Private methods

   Some of these sections won’t be always present. Sometimes it makes more
   sense to divide the class by functionality, but this is often a symptom that
   the class is trying to do too much.

   Rationale: While dividing classes into sections restricts flexibility
   somewhat, it leads to predictable code layout and makes orientation in
   classes much easier for programmers not familiar with the class code.

1. Do not put any empty lines after the beginning and before the end of the
   class body. Put empty lines between class sections.

1. Start sections with protected and private methods by `protected`/`private`
   method call, separated by an empty line from the following methods. Do not
   increase indentation of the following methods. Do not change method
   visibility explicitly by invoking the `public`/`protected`/`private` method
   with explicit method name(s).

1. When defining class methods, use the `def self.method_name` syntax:

   ```ruby
   class Math
     def self.sin(a)
       # …
     end

     def self.cos(a)
       # …
     end
   end
   ```

   Rationale: While explicit definition of the singleton class using `class <<
   self` is more logical (it corresponds to what the interpreter actually does
   internally) and allows to use class methods of the singleton class between
   `class << self` and `end`, the `self.` prefix makes singleton methods stand
   out. This is more useful in practice.

### Names

Use short and descriptive class names. Above all, they should make sense at the
place where the class instance is created or static methods called. Use
CamelCase for method names.

Usually, class names should contain a noun describing what the class instance
represents, possibly precised with few adjectives. If the primary purpose of
the class is executing some process, the class name should usually end with
“-er” (as in `FooImporter`, `BarExporter`, `BazScanner`, etc.)

### Module definitions

Rules for module definitions are basically the same as for class definitions.
