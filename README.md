rv2sa
=====
An application and a Ruby script that compose and decompose Scripts.rvdata2 of RPG Maker VX Ace.
rv2sa: A tool for decomposing and generating Scripts.rvdata2, which is read by the Game.exe of RPG Maker VX Ace.

## Usage
Execute from the command line.

### Decompose
(Decompose Scripts.rvdata2 and place it in the script folder)

`rv2sa -d Scripts.rvdata2 -o script`

The files included in Scripts.rvdata2 and Scripts.conf.rb will be automatically generated.

### Compose
(Combine the contents decomposed into the script folder back into Scripts.rvdata2)

`rv2sa -c script/Scripts.conf.rb -o Scripts.rvdata2`

#### Flags

* You can specify flags with the -f option. These flags can be used to control the addition of files in Scripts.conf.rb.
* By executing `rv2sa -f "debug,test"`, the two flags :debug and :test will be considered active when adding files in Scripts.conf.rb. For more details, refer to **How to write Scripts.conf.rb**.

#### Debug Information

* By specifying the -i option, rv2sa will automatically define variables.
    * `$rv2sa_path`: The working directory when rv2sa is executed will be assigned.
    * `$LOAD_PATH`: The `$LOAD_PATH` when rv2sa is executed will be added.

## How to write Scripts.conf.rb

All expressions possible in a Ruby script can be used.

### Adding Files

* By writing `add "filename"`, you can add a file to be included in Scripts.rvdata2.
* The "filename" part should be the **relative path** from Scripts.conf.rb with the **extension omitted**.
* You can use %Q() notation or here documents. When using here documents, you can use String.unindent to exclude leading indentation (so that the shallowest line's indentation becomes 0).

### Flags

* By writing `add "filename", :symbol`, the file will only be added if the symbol is specified as a flag when executing rv2sa.
* For example, if you specify a debug file as `add "file_for_debug", :debug`, it will be added when `-f debug` is specified during rv2sa execution, and not otherwise.
* Flags during add can be set in multiple by passing an array like `add "file", [:debug, :test]`. In this case, the file will be added if any of the flags are active.

### Importing Other Files

* By writing `import "filename"`, you can import another file.
* Regardless of the import source or destination, the contents of the file should be written with a relative path from that file.

## Preprocessing

When generating Scripts.rvdata2 through rv2sa, the contents of each file described in Scripts.conf.rb are rewritten based on specific notation. This can be used to remove code not used in the user environment beforehand.

### Notation

It is described in a format similar to the C/C++ preprocessor.

Anything written in the following format is considered a directive to the preprocessor.

`#identifier arguments`

* `#` and `identifier` must be contiguous.
* There must be at least one space between `identifier` and `arguments`.
* There may be spaces before `#`. However, if there are non-space characters before `#`, it is not considered a directive.

#### Types of Directives

##### #define

* `#define :name`
* `#define :name, value`

Defines a constant. If value is omitted, it is set to nil.

It can be used for branching directives and string replacement described later.

##### #undef

* Undefines a constant defined with #define.

##### #if-#else-#endif

* The range enclosed by `#if`-`#else`-`#end` is excluded from the code if the condition is not met.
* Nesting is possible.
* `#ifdef :name` is a shorthand for `#if defined(:name)`.
* `#ifndef :name` is true when `#ifdef :name` is false.
* `#elif` is equivalent to Ruby's elsif.
* `#else_ifdef :name` is equivalent to Ruby's elsif defined :name.
* `#else ifndef :name` is equivalent to Ruby's elsif (! defined :name).

##### #warning

* `#warning message`

Displays the message to standard error when executing rv2sa.

##### #error

* `#error message`

Causes rv2sa to fail and displays the message to standard error.

##### #include

* `#include filename`

Replaces this line with the contents of filename.

##### Others

The following expressions can be specified as arguments for directives.

* `defined(:name)` returns true if :name is #defined, otherwise false.
* `defined(:name, value)` returns the result of comparing the value of name with value using === if name is #defined.
* `defined_value(:name)` returns the value of a #defined constant. Returns nil if undefined.
    * It can be used with operators like `#if defined_value(:version) < 20`.

#### Flags

Flags specified when executing rv2sa are automatically `#define`d with a value of true.

#### String Replacement

During preprocessing, any string in the source code that starts with [A-Z] and consists only of [A-Z0-9_] will be replaced with its value if it is defined with `#define`.

* If you define `#define :VERSION, 10`, the `VERSION` in the source code will be replaced with `10`.
* It mechanically replaces everything, regardless of whether it is in expressions, comments, or strings in the source code.
* It does not perform replacements that require arguments like C/C++'s `#define JOIN(a, b) a##b`.

## Intended Usage of rv2sa

### Initial Setup
* Decompose the default generated Scripts.rvdata2 and expand it into a folder of your choice.

### During Updates
* Add/edit source code using a file manager.
* Generate Scripts.rvdata2 and then start the game.

# Notes
* The contents of the files should be written in UTF-8.
* Scripts.rvdata2 is read when Game.exe starts. It is not a problem to generate Scripts.rvdata2 after the editor starts. (As of RPG Maker VX Ace 1.02a)
* The editor does not reread Scripts.rvdata2 after it is loaded at startup. If you want to edit scripts in the editor's script editor, restart the editor. (As of RPG Maker VX Ace 1.02a)
* Opening the script editor or database in the editor will regenerate Scripts.rvdata2. The contents of the generated Scripts.rvdata2 will be what was loaded when the editor started. If you generate Scripts.rvdata2 externally after the editor starts and then open the script editor in the editor, the externally generated Scripts.rvdata2 will be discarded and revert to the contents at the time the editor started. (As of RPG Maker VX Ace 1.02a)  
  * To avoid this issue, you can use rvhook to call rv2sa every time Game.exe is started.

# Changelog
* 2.3.0 Added the ability to add debug information with the -i option.
* 2.2.0 Added a preprocessor.
* 2.1.0 Added `import`.
* 2.0.0 Changed the definition file to Scripts.conf.rb and revamped the script content.
* 1.3.0 Added the -l option.
* 1.2.0 Changed the file name to filename_index_id.rb.
* 1.1.0 Changed the script ID to a serial number during generation.
* 1.0.0 Initial version

# LICENSE

## rv2sa

The Ruby script of rv2sa is released under [NYSL](https://github.com/ctmk/rv2sa/blob/master/LICENSE).

## rv2sa.exe

rv2sa.exe is an exe version of rv2sa created with ocra.
It can be used in accordance with the license conditions of all the software it contains.

### Ruby

Ruby is released under Ruby's license.

https://www.ruby-lang.org/ja/

### ocra

ocra is released under the MIT License.

https://github.com/larsch/ocra

Copyright c 2009-2010 Lars Christensen

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

