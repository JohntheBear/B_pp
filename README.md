#The B Preprocessor

##A code template processor

### John Novak

###The what and why:
####What is this?
This is a code templating pre-processor. In my work I often have to write code which runs on multiple data files. The code has to be the same, but the files differ in subtle ways: the number of files, the format of the files, etc. Instead of writing one piece of code, then tweaking it to work for each case, I write a B template, and put all of the file-specific-subtleties into meta-data text files, and the B pre-processor handles all of the case specific tweaking.

####Why would you want this?
#####Readability
Everything is text files and python scripts. The output of the preprocessor looks like normal code that was written from scratch. I send code generated by the pre-processor to my boss all the time. It makes him happy because it looks like code he writes; It makes me happy because I don't have to be inefficient and write code his way.
#####Flexibility
The preprocessor uses simple ideas: replacement, iteration, files are treated as strings, the template boilerplate is handled with the same rules as everything else. You can do some crazy things with a little creativity.

####Why didn't you just learn to use one of the other template scripting languages?
Frankly, I wrote the first version of this over the course of two hours. I needed something which worked, and I needed it fast. It would have taken me longer to learn something new, than to make something new.

####Why B?
B comes before C! Most of my work coding is in C/C++. Now I can say things like, "I wrote you A B C... ++ template for that." Actually, no, thats a really limp joke. 

###The how:
Ultimately this is a python script, so you need python. The magic is in the templates. There is an example template included in the repository.

####Naming convention:
The B templates should be named [filename].B.[file ending] . For example Display.B.cpp will turn into Display.cpp post-processing. The upside of this is that syntax highlighting in your editor of choice will work for the template.

####Whats in a template:
There are four parts to a template:<br />
-GUIDE<br />
-TEMPLATE<br />
-ITERABLES<br />
-REFERENCES<br />
Each of these needs to be in capitals and prefaces with two '@' symbols. Example '@@TEMPLATE'. At the end of each part, it must be terminated with it's name in capitals followed by two '@' symbols. Example 'TEMPLATE@@'.

#####@@GUIDE
Here is where properties of the pre-processor can be set. There are currently three options which you can set: the number of passes that the compiler will take, the file delimeter, and the level indicator.<br />

Here are the current options and their defaults:<br />
> @Passes = 5<br />
> @Fdelimeter = %<br />
> @Levelindicator = !<br />
> @Verbose = 0<br />

The spaces around the '=' are *not* necessary.<br />

The option '@Passes' controls the number of passes taken which is explained in the section "Order of operations and multiple passes". The option '@Fdelimeter' controls the character or string which is read as file bookends, see section "Files". The option '@Levelindicator' is used to distinguish which pass the compiler should address each operation, as discussed in the section "Order of operations and multiple passes". The option '@Verbose' controls how much debug data is printed out. It should be an integer, larger numbers tend to spit out more information.

#####@@TEMPLATE
This is part of the template which turns into your code.

#####@@ITERABLES
This is the main meat and potatoes of the pre-processor. Iterables are list of sets of strings. Each iterable should be given in the form:
> @IterableName(prop1,prop2,prop3...):<br />
> string1-1 string1-2 string1-3 ...<br />
> string2-1 string2-2 string2-3 ...<br />
> ...<br />
> stringN-1 stringN-2 stringN-3 ...<br />

The list of properties in parentheses after the iterable name can be arbitrarily long. When an iterable is used in the TEMPLATE section, one of the properties needs to always be specified by following the iterable name with a period and the property name. Ex: "@IterableName.prop2@". Note the '@' bookends. When an iterable is used, the line it is used in will be repeated as many times as there are items in the iterable. If the same iterable is used more than once in the same line, every instance of the iterable is replaced at the same time. 

As an example. If I define an iterable:
> @Counting(Name,Value):<br />
> A 1<br />
> B 2<br />
> C 3<br />

And then I use it in the template:
> int @Counting.Name@ = @Counting.Value@;<br />

It will render as:
> int A = 1;<br />
> int B = 2;<br />
> int C = 3;<br />

There is also a special iterable '@i@', which will be replaced with the count of the iteration. So the above example could have been done as:
> Define iterable:
> > @Counting(Name):<br />
> > A<br />
> > B<br />
> > C<br />

> Use it:<br />
> > int @Counting.name@ = @i@;<br />

#####@@REFERENCES
Refernces are text strings which will be replaced. Each reference should be given in the form:<br />
> @RefName:<br />
> "string"<br />

Everywhere that @RefName@ is used it will be replaced with "string". Note that when a reference is used, it needs to have '@'s as bookends.

####Files
Other files can be included into a template as if they where part of the original. File names should have '%' bookends.

If just a filename is give, the entire file is loaded as if it was part of the original template.<br />
> %OtherFile.txt%  # Load the whole file<br />

You can specify parts of the file to load by following the filename with an argument in '[]' brackets. If one number is given in the brackets, only that line is loaded.<br />
> %OtherFile.txt[2]%  # Load line 2<br />

If two numbers are given, all of the lines between and including the two are loaded. Either number can be a ':', and then it will load up to the end of the file<br />
> %OtherFile.txt[2,5]%  # Load lines 2-5<br />
> %OtherFile.txt[2,:]%  # Load lines 2 to the end<br />
> %OtherFile.txt[:,5]%  # Load lines from the start to line 5<br />

If three numbers are given, seperated by commas, the lines between the first two, modulo the third are loaded. The ':' can still be used.<br />
> %OtherFile.txt[2,:,3]%  # Load every third line, from 2 to the end<br />

File replacement can be very usefull when defining iterables. I often have lists of files which will need to be loaded, and I can easily make an iterable out of them in a few steps:<br />
> At the command line:<br />
> > $ ls -1 \*.bin > filelist.txt <br />

> in the template:<br />
> > @@ITERABLES<br />
> > @Files(names):<br />
> > %filelist.txt%<br />

####Order of operations and multiple passes
The order in which things are processed is: file expansion, iterables, then reference replacement. Multiple passes are teken when processing a template, so it is possible to do creative things like use references in defining iterables. Currently the preprocessor take five passes, althougth this could be set as an option in GUIDE section. Priorites are set with '!'s. The more '!'s there are in front of it, the more important it is. Because the current max depth is five, the most '!'s you can use is four. No '!'s is lowest priority, so using '!' is entrirely optional. The '!' should be placed before the '@' bookends.

###EXAMPLE USE

I have included an example of a B template, and what you should get from it. This is an example taken directly from my work (sort of, I had to find something where I could make it public). The template is "Display.B.cpp".

To use the preprocessor:
> python B\_pp.py Display.B.cpp

or make B\_pp.py executable and run
> ./B\_pp.py Display.B.cpp

It will generate a file "Display.cpp", which can be checked against "Correct\_Display.cpp". There are a handful of other files lying around, and these are all used in the "Display.B.cpp" template.
