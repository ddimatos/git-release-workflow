# Creating a Project Page

How can we leverage the ansible-doc in Ansible modules to populate our 
documentation sites? How could we avoiding having to build a site that would 
have all people come to expect such as search, twisties, navigation and code blocks.

Today we will cover how we can do this using these tools to do just that. 
* Sphinx
* ansible-doc-extractor
* sphinx-rtd-theme

## Overview
- [Creating a Project Page](#creating-a-project-page)
   - [Virtual Environment](#virtual-environment)
   - [Clone Repository](#clone-repository)
   - [Configure Sphinx](#configure-sphinx)
   - [Restructured Text](#restructured-text)
   
  

## Virtual Environment

We will use a python virtual environment to install our tools and work from
that includes cloning the repositories. Virtual environments keep your existing
system dependencies from being impacted and lends well to migrating this 
process to CI/CD.

Using `tmp/html/` to stage all this:

```
$ mkdir -p /tmp/html
$ cd /tmp/html
$ python3 -m venv venv
$ . venv/bin/activate
$ pip install Sphinx
$ pip install ansible-doc-extractor
```

## Clone Repository

I will use the `IBM z/OS core collection` repository for this example. Because
this project already has created a GitHub page, there already exists a `docs/`
directory with the content (*.rst files), so we can clean it up. 

Clone repository and remove `docs/` directory:

```
$ git@github.com:ansible-collections/ibm_zos_core.git
$ cd ibm_zos_core
$ rm -rf docs/*
$ cd ..
```

## Configure Sphinx

![Sphinx](https://www.sphinx-doc.org/en/master/usage/quickstart.html) makes it
easy to create documentation, it supports plugins, cross-referencing and can 
output content in HTML, LaTeX, ePub, manual pages plain text and more. Here, 
we will only be discuss how to use Sphinx to generate HTML.

Initialize the Sphinx directory:
```
$ cd docs
$ sphinx-quickstart
```

You should be prompted to answer some questions, be sure to answer `Y` to a 
question about keeping the source and build separate, this will help us later
when we want to extract the generated HTML in `docs/source/build`.

Answer these interactive questions:
```
 > Separate source and build directories (y/n) [n]: Y
 > Project name: IBM Z Core Collection
 > Author name(s): IBM
 > Project release []: 1.0.0
 > Project language [en]: <press return take default>
```

## Restructured Text

Now that we everything is up and ready to go, we will need some restructured
text (*.rst) for Shpinx to read and generate HTML output. In the last step,
the `sphinx-quickstart` created the `source` directory and populated it with
some minimum requirements such as a `Makefile` and `source/` directory. 

//TODO: Provide a more detail explanation on how to structure RST and share
//some sample snippets. 

Shpinx created the `source/` directory, add a `modules` directory; this 
directory will be used for the generated reStructuredText when
`ansible-doc-extractor` reads the Ansible-doc and extracts into *.rst. This a
vary powerful feature, as it reduces the need to maintain docs in two places.
I highly recommend you invest into documenting the options, examples and 
return values, this will pay off in the long run.

Make the directory to store the generated `rst` files. 
```
mkdir -p docs/source/modules
```
Create an `index.rst`, this will serve as the master document for Sphinx; it
will create a index.html from this but more importantly it allows you to link
all your other `*.rst` files together and instruct Sphinx how to interpret 
them.  
```
vi docs/source/index.rst
```

Add this to the `index.rst`, it basically is telling Sphinx that it will be
able to find a number of `*.rst` files in the relative folder `modules/` and
after it renders them into HTML it should put under a heading called Contents
on the left side navigation. 
```
.. toctree::
   :maxdepth: 1
   :caption: Contents:
   :glob:

   modules/*
```

Lets select one module `../plugins/modules/zos_job_output.py` to extract as 
restructured into `source/modules` , such that ou will end up with
`source/modules/zos_job_output.rst`.
 
Then `make html` to have sphinx generate html based on the `index.rst` and 
module `source/modules/zos_job_output.rst` file, lastly view it with a browser.

```
$ cd docs
$ ansible-doc-extractor source/modules ../plugins/modules/zos_job_output.py
$ make html
$ open build/html/index.html
```

After you are done reviewing the content in the browser, clean up the generated
files. 
```
$ make clean
```

## Sphinx Themes
At this point you have probably noticed the generated HTML is not very appealing, this is where Shinx themes like
__Read the Docs__ will be very helpful. After installing the theme, we will configure Shinx to use the theme with a
few edits to the conf.py.

```
$ pip install sphinx-rtd-theme
$ vi source/conf.py

# Tell Sphinx about the extension
extensions = [
    "sphinx_rtd_theme",
]

# Tell Sphinx about the theme
html_theme = "sphinx_rtd_theme"

$ ansible-doc-extractor source/modules ../plugins/modules/zos_job_output.py
$ make html
$ open build/html/index.html
```

Now you should see a webdoc that looks quite a bit nicer, Read the Docs provides a great theme and service by offering
ths to the comunnity.

## Sphinx Templates

When the module's Ansible doc is extracted and converted to restructured text by `ansible-doc-extractor`, you can do
even better than the defaults; you can use jinja templates to instruct `ansible-doc-extractor` what is valid, how
it should be formated and what should be displayed. 

To use a template, you must configure Sphinx:
```
$ mkdir templates
$ vi source/conf.py

# Add the template path to the configuraton
templates_path = ['../templates']
```

Create a template; for now you can use the one that I have started working on, place it in the `templates` directory.
https://github.com/ansible-collections/ibm_zos_core/blob/dev/docs/templates/module.rst.j2.

In the commands below, noticed that I passed in the template as an argument to `ansible-doc-extractor` so that when it
extracts the restructured text, it will adhere to the jinja template. 

```
$ ansible-doc-extractor --template templates/module.rst.j2 source/modules ../plugins/modules/*.py
$ make html
$ open build/html/index.html
```

## Makefile Tips

After a while, you will find it tedious to run all these commands, you can edit the default make file to suite your
needs. At this point, without scripting, i could not figure out how to instruct `ansible-doc-extractor` to ignore 
certain files, thus it would try to read `__init_.py` and fail. To avoid this faiulre, I had the make file move it
to a temporary location and put it back when complete. 

Below are some edits I made to the makefile to simplify generation.

Add a clean:
```
clean: 
	rm -rf build
	echo "Deleted directory build/"

	rm -rf source/modules
	echo "Deleted directory source/modules"
	echo "Completed HTML text generation, run 'make ibm_zos_core'"
```

Add a ibm_zos_core:

```
ibm_zos_core:
	mkdir build
	mkdir -p source/modules
	mv ../plugins/modules/__init__.py ../plugins/modules/__init__.py.skip
	ansible-doc-extractor --template templates/module.rst.j2 source/modules ../plugins/modules/*.py
	echo "Completed restructured text generation, run 'make html'"
	mv ../plugins/modules/__init__.py.skip ../plugins/modules/__init__.py
```

Add a view:
```
view:
	open build/html/index.html
```

Update the catch-all:
```
%: Makefile
	@$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
	echo "Completed HTML text generation, run 'make view'"
```

# Git Page
After having run `make html` and susequent commands, you now have some static html in `build/html/*` that you will
want to host. Each GitHub project has a GitHub Pages feature designed to host static html and this is what we will
tailor our commands to.

```
# Create and orphan branch in your repo that has no history associated to it, this repo will be just for docs
$ git checkout --orphan gh-pages  

# Remove all the contents from the branch, you won't need it, we just need a bare repo
$ git rm -rf .

# This file tells GitHub Pages not to run the published files through Jekyll
$ touch .nojekyll
$ git add .nojekyll

# Init the branch with a commit
$ git commit -m "Initial gh-pages branch for documentation"

# Map the gh-pages repo to gh-pages/ dir and add it to a worktree
$ git checkout master
$ git worktree add gh-pages gh-pages

# Depending on if you have a prior build, you may not need to do this step.
$ cd docs
$ make clean
$ make ibm_zos_core
$ make html

# Copy all the html files generated with rysnc (a - recursive, v -verbose) to the `gh-pages/` directory. 
# This should make sense now why we cloned master and ran the make commands to get the html
$ rsync -av docs/build/html/ gh-pages/

# Commit it to gh-pages repo
$ cd gh-pages
$ rm -rf .DS_Store
$ git add .
$ git commit -m "Initial documentation commit"
$ git push -u origin gh-pages
$ cd ..
```

TIP: I find it helpful to edit the python code with some print statements to view the html genertion as well as
doc-extrator (TODO: share these edits here)

TODO: Eplain how to do this with without git worktrees, might be easier for some even though more mechanical.

# Git Page Update

Updates (regenerating the html) should be pretty easy the intial documentation releasae. 

``
# Since the gh-pages is there to manage doc, you can delete prior builds
$ rm -rf docs/build

# Build the new HTML content
$ cd docs
$ make clean
$ make ibm_zos_core
$ make html

# Copy all the html files generated with rysnc (a - recursive, v -verbose) to the `gh-pages/` directory. 
# This should make sense now why we cloned master and ran the make commands to get the html
$ rsync -av docs/build/html/ gh-pages/

# Commit it to gh-pages repo
$ cd gh-pages
$ rm -rf .DS_Store
$ git add .
$ git commit -m "Initial documentation commit"
$ git push -u origin gh-pages
$ cd ..
``


# Trials
I tried to use and MD to RST converter, it did not fair well with any of the links or indentation but I did not put
much effort into this:
```
$ pip install mdToRst
$ mdToRst README.md >README.rst
```

Not sure why i tried this :) :
```
pip install sphinx-jinja
```