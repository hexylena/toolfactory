toolfactory_2
=============

This is an upgrade to the tool factory but with added parameters 
(optionally editable in the generated tool form - otherwise fixed) and 
multiple input files.

Specify any number of parameters - well at 
least up to the limit of your patience with repeat groups.

Parameter values supplied at tool generation time are defaults and 
can be optionally editable by the user - names cannot be changed once
a tool has been generated.

If not editable, they act as hidden parameters passed to the script 
and are not editable on the tool form.

Note! There will be Galaxy default sanitization for all 
user input parameters which your script may need to dance around.

Any number of input files can be passed to your script, but of course it
has to deal with them. Both path and metadata name are supplied either in the environment 
(bash/sh) or as command line parameters (python,perl,rscript) that need to be parsed and
dealt with in the script. This is complicated by the common use case of needing file names
for (eg) column headers, as well as paths. Try the examples are show on the tool factory 
form to see how Galaxy file and user supplied parameter values can be recovered in each 
of the 4 scripting environments supported.

Best way to deal with multiple outputs is to let the tool factory generate an HTML
page for your users. It automagically lays out pdf images as thumbnail galleries
and can have separate results sections gathering all similarly prefixed files, such as
a Foo section taking text and results from text (foo_whatever.log) and 
artifacts (eg foo_MDS_plot.pdf) file names. All artifacts are linked for download.
A copy of the actual script is provided for provenance - be warned, it exposes
real file paths.

![example run](/images/dynamicScriptTool.png)

tldr;

```

﻿# WARNING before you start
# Install this tool on a private Galaxy ONLY
# Please NEVER on a public or production instance
# updated august 2014 by John Chilton adding citation support
#
# updated august 8 2014 to fix bugs reported by Marius van den Beek
# please cite the resource at
http://bioinformatics.oxfordjournals.org/cgi/reprint/bts573?ijkey=lczQh1sWrMwdYWJ&keytype=ref
# if you use this tool in your published work.

*Short Story*

This is an unusual Galaxy tool capable of generating new Galaxy tools.
It works by exposing *unrestricted* and therefore extremely dangerous scripting
to all designated administrators of the host Galaxy server, allowing them to
run scripts in R, python, sh and perl over multiple selected input data sets,
writing a single new data set as output.

*Differences between TF2 and the original Tool Factory*

1. TF2 (this one) allows any number of either fixed or user-editable parameters to be defined
for the new tool. If these are editable, the user can change them but otherwise, they are passed
as fixed and invisible parameters for each execution. Obviously, there are substantial security
implications with editable parameters, but these are always sanitized by Galaxy's inbuilt 
parameter sanitization so you may need to "unsanitize" characters - eg translate all "__lt__" 
into "<" for certain parameters where that is needed. Please practise safe toolshed.

2. Any number of (the same datatype) of input files may be defined.

These changes substantially complicate the way your supplied script is supplied with
all the new and variable parameters. Examples in each scripting language are shown
in the tool help

*Automated outputs in named sections*

If your script writes to the current directory path, arbitrary mix of (eg)
pdfs, tabular analysis results and run logs,the tool factory can optionally
auto-generate a linked Html page with separate sections showing a thumbnail
grid for all pdfs and the log text, grouping all artifacts sharing a file
name and log name prefix::

 eg: if "foo.log" is emitted then *all* other outputs matching foo_* will
 all be grouped together - eg
 foo_baz.pdf
 foo_bar.pdf and
 foo_zot.xls
 would all be displayed and linked in the same section with foo.log's contents
 - to form the "Foo" section of the Html page.  Sections appear in alphabetic
 order and there are no limits on the number of files or sections.

*Automated generation of new Galaxy tools for installation into any Galaxy*

Once a script is working correctly, this tool optionally generates a
new Galaxy tool, effectively freezing the supplied script into a new,
ordinary Galaxy tool that runs it over one or more input files selected by
the user. Generated tools are installed via a tool shed by an administrator
and work exactly like all other Galaxy tools for your users.

If you use the Html output option, please ensure that sanitize_all_html is
set to False and uncommented in universe_wsgi.ini - it should show::

 # By default, all tool output served as 'text/html' will be sanitized
 sanitize_all_html = False

This opens potential security risks and may not be acceptable for public
sites where the lack of stylesheets may make Html pages damage onlookers'
eyeballs but should still be correct.


*More Detail*

To use the ToolFactory, you should have prepared a script to paste into a
text box, and a small test input example ready to select from your history
to test your new script.

There is an example in each scripting language on the Tool Factory form. You
can just cut and paste these to try it out - remember to select the right
interpreter please. You'll also need to create a small test data set using
the Galaxy history add new data tool.

If the script fails somehow, use the "redo" button on the tool output in
your history to recreate the form complete with broken script. Fix the bug
and execute again. Rinse, wash, repeat.

Once the script runs sucessfully, a new Galaxy tool that runs your script
can be generated. Select the "generate" option and supply some help text and
names. The new tool will be generated in the form of a new Galaxy datatype
- toolshed.gz - as the name suggests, it's an archive ready to upload to a
Galaxy ToolShed as a new tool repository.

Once it's in a ToolShed, it can be installed into any local Galaxy server
from the server administrative interface.

Once the new tool is installed, local users can run it - each time, the script
that was supplied when it was built will be executed with the input chosen
from the user's history. In other words, the tools you generate with the
ToolFactory run just like any other Galaxy tool,but run your script every time.

Tool factory tools are perfect for workflow components. One input, one output,
no variables.

*To fully and safely exploit the awesome power* of this tool,
Galaxy and the ToolShed, you should be a developer installing this
tool on a private/personal/scratch local instance where you are an
admin_user. Then, if you break it, you get to keep all the pieces see
https://bitbucket.org/fubar/galaxytoolfactory/wiki/Home

** Installation **
This is a Galaxy tool. You can install it most conveniently using the
administrative "Search and browse tool sheds" link. Find the Galaxy Main
toolshed at https://toolshed.g2.bx.psu.edu/ and search for the toolfactory
repository. Open it and review the code and select the option to install it.

(
If you can't get the tool that way, the xml and py files here need to be
copied into a new tools
subdirectory such as tools/toolfactory Your tool_conf.xml needs a new entry
pointing to the xml
file - something like::

  <section name="Tool building tools" id="toolbuilders">
    <tool file="toolfactory/rgToolFactory.xml"/>
  </section>

If not already there (I just added it to datatypes_conf.xml.sample),
please add:
<datatype extension="toolshed.gz" type="galaxy.datatypes.binary:Binary"
mimetype="multipart/x-gzip" subclass="True" />
to your local data_types_conf.xml.
)

Of course, R, python, perl etc are needed on your path if you want to test
scripts using those interpreters. Adding new ones to this tool code should
be easy enough. Please make suggestions as bitbucket issues and code. The
HTML file code automatically shrinks R's bloated pdfs, and depends on
ghostscript. The thumbnails require imagemagick .

* Restricted execution *
The tool factory tool itself will then be usable ONLY by admin users -
people with IDs in admin_users in universe_wsgi.ini **Yes, that's right. ONLY
admin_users can run this tool** Think about it for a moment. If allowed to
run any arbitrary script on your Galaxy server, the only thing that would
impede a miscreant bent on destroying all your Galaxy data would probably
be lack of appropriate technical skills.

*What it does* This is a tool factory for simple scripts in python, R and
perl currently. Functional tests are automatically generated. How cool is that.

LIMITED to simple scripts that read one input from the history. Optionally can
write one new history dataset, and optionally collect any number of outputs
into links on an autogenerated HTML index page for the user to navigate -
useful if the script writes images and output files - pdf outputs are shown
as thumbnails and R's bloated pdf's are shrunk with ghostscript so that and
imagemagik need to be available.

Generated tools can be edited and enhanced like any Galaxy tool, so start
small and build up since a generated script gets you a serious leg up to a
more complex one.

*What you do* You paste and run your script, you fix the syntax errors and
eventually it runs. You can use the redo button and edit the script before
trying to rerun it as you debug - it works pretty well.

Once the script works on some test data, you can generate a toolshed compatible
gzip file containing your script ready to run as an ordinary Galaxy tool in
a repository on your local toolshed. That means safe and largely automated
installation in any production Galaxy configured to use your toolshed.

*Generated tool Security* Once you install a generated tool, it's just
another tool - assuming the script is safe. They just run normally and their
user cannot do anything unusually insecure but please, practice safe toolshed.
Read the fucking code before you install any tool. Especially this one -
it is really scary.

If you opt for an HTML output, you get all the script outputs arranged
as a single Html history item - all output files are linked, thumbnails for
all the pdfs. Ugly but really inexpensive.

Patches and suggestions welcome as bitbucket issues please?

copyright ross lazarus (ross stop lazarus at gmail stop com) May 2012

all rights reserved
Licensed under the LGPL if you want to improve it, feel free
https://bitbucket.org/fubar/galaxytoolfactory/wiki/Home

Material for our more enthusiastic and voracious readers continues below -
we salute you.

**Motivation** Simple transformation, filtering or reporting scripts get
written, run and lost every day in most busy labs - even ours where Galaxy is
in use. This 'dark script matter' is pervasive and generally not reproducible.

**Benefits** For our group, this allows Galaxy to fill that important dark
script gap - all those "small" bioinformatics tasks. Once a user has a working
R (or python or perl) script that does something Galaxy cannot currently do
(eg transpose a tabular file) and takes parameters the way Galaxy supplies
them (see example below), they:

1. Install the tool factory on a personal private instance

2. Upload a small test data set

3. Paste the script into the 'script' text box and iteratively run the
insecure tool on test data until it works right - there is absolutely no
reason to do this anywhere other than on a personal private instance.

4. Once it works right, set the 'Generate toolshed gzip' option and run
it again.

5. A toolshed style gzip appears ready to upload and install like any other
Toolshed entry.

6. Upload the new tool to the toolshed

7. Ask the local admin to check the new tool to confirm it's not evil and
install it in the local production galaxy



**Parameter passing and file inputs**

Your script will receive up to 3 named parameters
INPATHS is a comma separated list of input file paths
INNAMES is a comma separated list of input file names in the same order
OUTPATH is optional if a file is being generated, your script should write there
Your script should open and write files in the provided working directory if you are using the Html
automatic presentation option.

Python script command lines will have --INPATHS and --additional_arguments etc. to make it easy to use argparse

Rscript will need to use commandArgs(TRUE) - see the example below - additional arguments will
appear as themselves - eg foo="bar" will mean that foo is defined as "bar" for the script.

Bash and sh will see any additional parameters on their command lines and the 3 named parameters
in their environment magically - well, using env on the CL

***python***::

 # argparse for 3 possible comma separated lists
 # additional parameters need to be parsed !
 # then echo parameters to the output file
 import sys
 import argparse
 argp=argparse.ArgumentParser()
 argp.add_argument('--INNAMES',default=None)
 argp.add_argument('--INPATHS',default=None)
 argp.add_argument('--OUTPATH',default=None)
 argp.add_argument('--additional_parameters',default=[],action="append")
 argp.add_argument('otherargs', nargs=argparse.REMAINDER)
 args = argp.parse_args()
 f= open(args.OUTPATH,'w')
 s = '### args=%s\n' % str(args)
 f.write(s)
 s = 'sys.argv=%s\n' % sys.argv
 f.write(s) 
 f.close()



***Rscript***::

 # tool factory Rscript parser suggested by Forester
 # http://www.r-bloggers.com/including-arguments-in-r-cmd-batch-mode/
 # additional parameters will appear in the ls() below - they are available
 # to your script
 # echo parameters to the output file
 ourargs = commandArgs(TRUE)
 if(length(ourargs)==0){
    print("No arguments supplied.")
 }else{
    for(i in 1:length(ourargs)){
         eval(parse(text=ourargs[[i]]))
    }
 sink(OUTPATH)
 cat('INPATHS=',INPATHS,'\n')
 cat('INNAMES=',INNAMES,'\n')
 cat('OUTPATH=',OUTPATH,'\n')
 x=ls()
 cat('all objects=',x,'\n')
 sink()
 }
 sessionInfo()
 print.noquote(date())


***bash/sh***::

 # tool factory sets up these environmental variables
 # this example writes those to the output file
 # additional params appear on command line
 if [ ! -f "$OUTPATH" ] ; then
    touch "$OUTPATH"
 fi
 echo "INPATHS=$INPATHS" >> "$OUTPATH"
 echo "INNAMES=$INNAMES" >> "$OUTPATH"
 echo "OUTPATH=$OUTPATH" >> "$OUTPATH"
 echo "CL=$@" >> "$OUTPATH"

***perl***::

 (my $INPATHS,my $INNAMES,my $OUTPATH ) = @ARGV;
 open(my $fh, '>', $OUTPATH) or die "Could not open file '$OUTPATH' $!";
 print $fh "INPATHS=$INPATHS\n INNAMES=$INNAMES\n OUTPATH=$OUTPATH\n";
 close $fh;
 


Galaxy as an IDE for developing API scripts
If you need to develop Galaxy API scripts and you like to live dangerously,
please read on.

Galaxy as an IDE?
Amazingly enough, blend-lib API scripts run perfectly well *inside*
Galaxy when pasted into a Tool Factory form. No need to generate a new
tool. Galaxy+Tool_Factory = IDE I think we need a new t-shirt. Seriously,
it is actually quite useable.

Why bother - what's wrong with Eclipse
Nothing. But, compared with developing API scripts in the usual way outside
Galaxy, you get persistence and other framework benefits plus at absolutely
no extra charge, a ginormous security problem if you share the history or
any outputs because they contain the api script with key so development
servers only please!

Workflow
Fire up the Tool Factory in Galaxy.

Leave the input box empty, set the interpreter to python, paste and run an
api script - eg working example (substitute the url and key) below.

It took me a few iterations to develop the example below because I know
almost nothing about the API. I started with very simple code from one of the
samples and after each run, the (edited..) api script is conveniently recreated
using the redo button on the history output item. So each successive version
of the developing api script you run is persisted - ready to be edited and
rerun easily. It is ''very'' handy to be able to add a line of code to the
script and run it, then view the output to (eg) inspect dicts returned by
API calls to help move progressively deeper iteratively.

Give the below a whirl on a private clone (install the tool factory from
the main toolshed) and try adding complexity with few rerun/edit/rerun cycles.

Eg tool factory api script
import sys
from blend.galaxy import GalaxyInstance
ourGal = 'http://x.x.x.x:xxxx'
ourKey = 'xxx'
gi = GalaxyInstance(ourGal, key=ourKey)
libs = gi.libraries.get_libraries()
res = []
# libs looks like
# u'url': u'/galaxy/api/libraries/441d8112651dc2f3', u'id':
u'441d8112651dc2f3', u'name':.... u'Demonstration sample RNA data',
for lib in libs:
    res.append('%s:\n' % lib['name'])
    res.append(str(gi.libraries.show_library(lib['id'],contents=True)))
outf=open(sys.argv[2],'w')
outf.write('\n'.join(res))
outf.close()

**Attribution**
Creating re-usable tools from scripts: The Galaxy Tool Factory
Ross Lazarus; Antony Kaspi; Mark Ziemann; The Galaxy Team
Bioinformatics 2012; doi: 10.1093/bioinformatics/bts573

http://bioinformatics.oxfordjournals.org/cgi/reprint/bts573?ijkey=lczQh1sWrMwdYWJ&keytype=ref

**Licensing**
Copyright Ross Lazarus 2010
ross lazarus at g mail period com

All rights reserved.

Licensed under the LGPL

**Obligatory screenshot**

http://bitbucket.org/fubar/galaxytoolmaker/src/fda8032fe989/images/dynamicScriptTool.png


```

