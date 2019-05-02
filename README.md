# Myaamori's Script Collection

## Merge Scripts

### Overview

![overview](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_overview.png)

Merge Scripts is an advanced bi-directional script merger.
It allows you to define a set of imports in an ASS file, which can then be used to import and de-import other files, as well as export any changes to the original files.
This is particularly useful when you are using a VCS such as git to store your files in a modular layout, where e.g. dialogue, songs and typesetting are stored in separate files and where the filenames don't change.
In such a scenario, you only need to define the imports once, after which all scripts can be re-merged in a single click anytime something is changed.
No more manually having to merge and shift songs every time you change something in the OP!
The ability to export changes also means that the modular structure can be kept, while still allowing for e.g. easy QC from a single file.

### Importing files

To import files you must first create an import definition.
Merge Scripts can do this for you using the **Add file to include** option.
Simply select the file or files that you wish to include, and import definitions will be added for you at the end of the file.

![overview](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_importdef.png)

This script contains import definitions for three files: `shoujo kageki 03.ass`, `insert 03.ass`, and `signs 03.ass`.
In order to actually import the files, we use the **Import all external files** option.
This will read the files specified in the import definitions and add their styles and dialogue lines to the current script.

![after import](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_postimport.png)

Every style will be prefixed with an identifier to keep track of what file the line came from.
Files can be imported selectively using **Import selected external file(s)**, which will only import files corresponding to the currently selected import definitions.

Once files have been imported, they can be de-imported to get back the original file prior to importing using the **Remove all external files** option.
This will delete all lines that belong to imported files.
Much like with importing, you can remove files selectively using **Remove selected external file(s)**, which will only remove the file corresponding to the currently selected files (you may select either the import definition or any line from the file you wish to remove).

### Exporting changes

The **Export changes** option allows you to export any changes you've made to the lines to their original files.
You will get a warning showing what files will be overwritten upon export.
Note that the script will currently *not* export script properties, but will leave the Script Info and Aegisub Project Garbage sections as they were in the source file.
Export changes will keep any extradata generated by automations intact.

### Shifted imports

![shifted import](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_shiftedimport.png)

Shifted imports automatically shift the lines in the imported file so that the start time of an synchronization line in the external file matches the start time of the import definition.
Run **Add synchronization line for shifted imports** on the file to import to add a new synchronization line at the current video time, and **Add file to include (shifted)** to add a shifted import definition at the current video time.

When exporting changes to a shifted file, the timings will be shifted back to match the original position of the synchronization line.
This means that shifting the lines will not affect the timing in the source file, however relative adjustments to individual lines such as fixed scene bleeds will be exported.

### Conflicting script properties on import

![conflicting properties](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_conflict.png)

When importing files, Merge Scripts will set script properties such as the resolution of the current script to match the imported files.
However, if there are conflicting values among the imported scripts, Merge Scripts will warn about this and let you choose which value to use.
If you hover over the dropdown box you can see the corresponding value from each individual file.
Note that **Merge Scripts will not attempt to resample the lines based on your selection**.
Thus, in the event of a conflict it is usually best to correct the offending file first.

### Generating a release version

Since you may not want to release the script publicly with all styles prefixed and the import definitions included (not that it would affect the viewing experience in any way), Merge Scripts also provides the **Generate release candidate** option.
This will remove the prefixes from all styles, remove unused styles, and remove all commented lines.
It will also warn about conflicting style names.

## Sub Digest

### Overview

```
subdigest -i script.ass --selection-set style "Default|Alt" --keep-selected --remove-unused-styles --sort-field start ASC
```

Sub Digest is a utility for quick processing of ASS files.
At the basic level it can do e.g. sorting, simple replacement with regex, and modification of values using Python expressions.
However, most of its power comes from the use of *selections*, which make it possible to operate on just a subset of the file.

Sub Digest makes use of [python-ass](https://github.com/chireiden/python-ass), and inherits many of its conventions, including field names.
It is highly recommended to familiarize yourself with python-ass first before making use of Sub Digest.

### Installation

Install Sub Digest with pip:

```
pip install git+https://github.com/TypesettingTools/Myaamori-Aegisub-Scripts/#subdirectory=scripts/sub-digest
```

This will add a `subdigest` executable along with the `subdigest.py` Python module.
If the executable is not in your path, you can run Sub Digest as `python -m subdigest --help`.

### Workflow

The arguments to Sub Digest are processed as consecutive commands, with the output of one command being fed as the input to the next.
If you run e.g. `--sort-field start ASC --sort-field style ASC`, it will first sort by start time and then by style name.

By default Sub Digests works on the events section of the file, which contains the dialogue lines.
You can use the `--use-styles` argument to switch to the styles section instead, so that any following arguments will modify the styles section.
Use `--use-events` to switch back.

### Selections

By default, all lines are treated as selected, meaning that most commands such as `--sort-field` will operate on all lines (in the current section).
If you wish to only operate on a subset of lines, use the `--selection-*` commands.
E.g. `--selection-set style "^Default"` will set the selection to all lines whose style begins with "Default", and `--selection-intersect TYPE "Comment"` will intersect the current selection with all commented lines.
To reset the selection, and thus select all lines again, use `--selection-clear`.
Operations that support selections include `--sort-*`, `--modify-*`, `--keep-selected` and `--remove-selected`.

### Expressions

Some commands have alternative versions that take arbitrary Python expressions.
In these expressions, the current line will be available as `_`.
There are also a number of functions available to make it easier to modify the start and end times of a line, which are represented as `datetime.timedelta` objects, namely: `mins(x)`, `secs(x)` and `millis(x)`, which take a float and return a corresponding `timedelta` object.
For instance, in order to add a lead-in of 100 ms to all lines, you can use `--modify-expr start _.start - secs(0.1)`.

### Use from Python (interactive session)

Sub Digest also provides a `subdigest.py` module which can be imported from Python.
This allows for the possibility of interactively editing an ASS file from the Python REPL.

```
$ python
>>> import ass
>>> import subdigest
>>> with open('script.ass') as f:
...   subs = subdigest.Subtitles(ass.parse(f))
...
>>> subs.
subs.get_field(                 subs.move_selection(            subs.selection                  subs.selection_intersect_expr(  subs.sort_expr(
subs.keep_selected(             subs.remove_all_tags(           subs.selection_add(             subs.selection_set(             subs.sort_field(
subs.merge_file(                subs.remove_selected(           subs.selection_add_expr(        subs.selection_set_expr(        subs.sub_file
subs.modify_expr(               subs.remove_unused_styles(      subs.selection_clear(           subs.selection_subtract(        subs.use_events(
subs.modify_field(              subs.section                    subs.selection_intersect(       subs.selection_subtract_expr(   subs.use_styles(
>>> subs.selection_set("style", "^Default")
<subdigest.Subtitles object at 0x7f9ba4d49e48>
>>> subs.modify_field("text", "-chan", "")
<subdigest.Subtitles object at 0x7f9ba4d49e48>
>>> with open('output.ass', 'w') as f:
...   subs.dump_file(f)
```

All `ass.document.Document` properties and methods are available directly from the `subdigest.Subtitles` object, including the `events` and `styles` objects.

Note that at the current point in time most operations are performed in-place, meaning that there is no way to undo a change.

### Examples

Get the text of dialogue lines as plain text, with override tags and comments removed:
```
subdigest -i script.ass --remove-all-tags --get-field text
```

Change all instances of `-chan` and `-san` to `{**-chan}` and `{**-san}`, respectively, for use with [Daiz's Autoswapper](https://github.com/Daiz/AegisubMacros/#autoswapperlua---autoswapper):
```
subdigest -i script.ass --in-place --modify-field text "(-(chan|san))" "{**\1}"
```

Extract only the dialogue lines from a muxed script:
```
subdigest -i script.ass -o dialogue.ass --selection-set style "Default|Alt" --keep-selected --remove-unused-styles --sort-field start ASC
```

Use together with [Prass](https://github.com/tp7/Prass/):
```
subdigest -i script.ass --selection-set style "Default" --keep-selected | prass tpp - --lead-in 100 --lead-out 200 | subdigest --sort-field start ASC
```

Merge files:
```
subdigest -i dialogue.ass --merge-file typesetting.ass --merge-file OP.ass
```

Remove all lines within the first 5 minutes:
```
subdigest -i dialogue.ass --set-selection-expr "_.start < mins(5)" --remove-selected
```
