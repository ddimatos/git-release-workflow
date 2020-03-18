| [Home](README.md) ▸ **Creating a Project Page** |
|-----|
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
   - [Sphinx Themes](#sphinx-themes)
   - [Makefile Tips](#makefile-tips)
   - [Git Page](#git-page)
- [Pro-tip](#pro-tip)
- [Update GitHub Page](#update-gitHub-page)
- [Research](#research)


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
At this point you have probably noticed the generated HTML is not very 
appealing, this is where Sphinx themes like
![Read the Docs](https://readthedocs.org/) will be very helpful. Sphinx
can generate HTML from a theme like **sphinx-rtd-theme**, see this 
![reference](https://docs.readthedocs.io/en/stable/intro/getting-started-with-sphinx.html)
for more or their ![GitHub](https://github.com/readthedocs/sphinx_rtd_theme)

Install the theme:
```
$ pip install sphinx-rtd-theme
```

Note: If you are using the `ibm_zos_core` repository, you will find all this is
already configured and for reference. 

Configure Sphinx to use the theme `sphinx_rtd_theme` in `source/conf.py`
```
$ vi source/conf.py
```

Edit `source/conf.py` with these added configurations:
```
extensions = [
    "sphinx_rtd_theme",
]

html_theme = "sphinx_rtd_theme"
```

Repeat the HTML generation with the theme and view the updated build.
```
$ ansible-doc-extractor source/modules ../plugins/modules/zos_job_output.py
$ make html
$ open build/html/index.html
```

Now you should see a documentation that looks nicer. `Read the Docs` provides 
a great theme and service by offering this to the community. 

## Sphinx Templates

When the module's Ansible-doc is extracted and converted to restructured text 
by `ansible-doc-extractor`, you can go one step further and control how Sphinx
will author the HTML for the modules. Using a
 ![Jinja](https://jinja.palletsprojects.com/en/master/templates/) template you
can to logically instruct the generated HTML layout. 

Note: If you are using the `ibm_zos_core` repository, you will find all this is
already configured and for reference. 

To use a template, configure Sphinx:
```
$ mkdir templates
$ vi source/conf.py
```

# Add the template path to the configuration `source/conf.py`
```
templates_path = ['../templates']
```

In the `docs/templates/` directory, create a
![Jinja](https://jinja.palletsprojects.com/en/master/templates/) template. You
can use the the one in the `ibm_zos_core` repository. Copy the `module.rst.j2`
template into your `docs/templates/` directory 
https://github.com/ansible-collections/ibm_zos_core/blob/dev/docs/templates/module.rst.j2.

In the commands below, notice that I passed the template as an argument 
to `ansible-doc-extractor` so that when it extracts the restructured text, it 
will adhere to the 
![Jinja](https://jinja.palletsprojects.com/en/master/templates/) template. 

```
$ ansible-doc-extractor --template templates/module.rst.j2 source/modules ../plugins/modules/*.py
$ make html
$ open build/html/index.html
```
Tip: You can use the ![Jinja template](https://github.com/xlab-si/ansible-doc-extractor/blob/master/src/ansible_doc_extractor/templates/module.rst.j2)
that comes with ansible-doc-extractor also as a starting point to your own. 

## Makefile Tips

If you find it tedious to run the various `make` commands, you can customize 
`make` to automate these steps.

Note: I could not figure out how to instruct `ansible-doc-extractor` to ignore 
certain files, thus it would try to read `__init_.py` and fail. To avoid this
failure, I had `make` move it to a temporary location and put it back when
`make ibm_zos_core` completed.

Below are some edits I made to the makefile to simplify generation.

Add clean:
```
clean: 
	rm -rf build
	echo "Deleted directory build/"

	rm -rf source/modules
	echo "Deleted directory source/modules"
	echo "Completed HTML text generation, run 'make ibm_zos_core'"
```

Add ibm_zos_core:
```
ibm_zos_core:
	mkdir build
	mkdir -p source/modules
	mv ../plugins/modules/__init__.py ../plugins/modules/__init__.py.skip
	ansible-doc-extractor --template templates/module.rst.j2 source/modules ../plugins/modules/*.py
	echo "Completed restructured text generation, run 'make html'"
	mv ../plugins/modules/__init__.py.skip ../plugins/modules/__init__.py
```

Add view:
```
view:
	open build/html/index.html
```

Update catch-all:
```
%: Makefile
	@$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
	echo "Completed HTML text generation, run 'make view'"
```

# Git Page
After you have generated the HTML (`build/html/*`) , you should be thinking
about hosing. GitHub offers every repository a
![GitHub page](https://help.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
that can host HTML.

The following commands are tailored for GitHub pages.

Create and orphan branch in your repo that has no history associated to it, 
this repo will be just for docs.
```
$ git checkout --orphan gh-pages  
```

Remove all the contents from the branch, we only need an empty branch:
```
$ git rm -rf .
```

Add a `.nojekyll` file, this instructs GitHub Pages not to run the published
files through Jekyll:
```
$ touch .nojekyll
$ git add .nojekyll
```

Init the branch with a commit:
```
$ git commit -m "Initial gh-pages branch for documentation"
```

Map the gh-pages repo to gh-pages/ dir and add it to 
a ![worktree](https://git-scm.com/docs/git-worktree):
```
$ git checkout master
$ git worktree add gh-pages gh-pages
```

Generate HTML:
```
$ cd docs
$ make clean
$ make ibm_zos_core
$ make html
```

Copy the HTML files generated with rysnc (a - recursive, v -verbose) to 
the `gh-pages/` directory. By now it should be obvious as to why we cloned 
the `master` branch and ran the make commands to generate the HTML.
```
$ rsync -av docs/build/html/ gh-pages/
```

Commit the generated HTML to gh-pages branch:
```
$ cd gh-pages
$ rm -rf .DS_Store
$ git add .
$ git commit -m "Initial documentation commit"
$ git push -u origin gh-pages
$ cd ..
```

## Pro-tip

I found it helpful to edit the python code parsing the  ReStructuredText with 
some added print statements so that you can view where an error might be
occurring during HTML generation as well as in `ansible-doc-extractor`. 


Python File: <path-to-virtual-python>/venv/lib/python3.8/site-packages/ansible_doc_extractor/cli.py
Line: 29
Edit: print("rst_ify {}".format(text))
```
def rst_ify(text):
    print("rst_ify {}".format(text))
    t = _ITALIC.sub(r"*\1*", text)
    t = _BOLD.sub(r"**\1**", t)
    t = _MODULE.sub(r":ref:`\1 <\1_module>`", t)
    t = _LINK.sub(r"`\1 <\2>`_", t)
    t = _URL.sub(r"\1", t)
    t = _CONST.sub(r"``\1``", t)
    t = _RULER.sub(r"------------", t)

    return t
```

Python File: <path-to-virtual-python>/venv/lib/python3.8/site-packages/jinja2/filters.py
Line: 654
Edit: print("Indented lines {}".format(s))
```
    if blank:
        rv = (newline + indention).join(s.splitlines())
    else:
        print("Indented lines {}".format(s))
        lines = s.splitlines()
        rv = lines.pop(0)

        if lines:
            rv += newline + newline.join(
                indention + line if line else line for line in lines
            )
```

Python File: <path-to-virtual-python>/venv/lib/python3.8/site-packages/jinja2/runtime.py
Line: 679
Edit: print("Arguments {}".format(*arguments))
```
    def _invoke(self, arguments, autoescape):
        """This method is being swapped out by the async implementation."""
        print("Arguments {}".format(*arguments))
        rv = self._func(*arguments)
        if autoescape:
            rv = Markup(rv)
        return rv
```

# Update GitHub Page

Updating the static HTML in GitHub pages is pretty easy after the initial
setup and configuration. 

Note: I documented but not tested the commands below.

Since the `gh-pages` branch is only in place to manage doc, you don't need to 
keep prior commits, so delete the content.
```
$ rm -rf docs/build
```

Generate updated HTML:
```
$ cd docs
$ make clean
$ make ibm_zos_core
$ make html
```

Copy the HTML files generated with rysnc (a - recursive, v -verbose) to 
the `gh-pages/` directory. By now it should be obvious as to why we cloned 
the `master` branch and ran the make commands to generate the HTML.

```
$ rsync -av docs/build/html/ gh-pages/
```

Commit HTML to the `gh-pages` branch:
```
$ cd gh-pages
$ rm -rf .DS_Store
$ git add .
$ git commit -m "Updated documentation version"
$ git push -u origin gh-pages
$ cd ..
```

# Research

Things I tried, things that did not work and items lined up for research. 

## Tested
I tried to use this MD to RST conversion utility and it did not correctly
generate the RST files correctly, particularly URLs.
```
$ pip install mdToRst
$ mdToRst README.md >README.rst
```

Not sure why i installed
![sphinx-jinja](https://pypi.org/project/sphinx-jinja/), but don't have any 
comments. 

```
pip install sphinx-jinja
```

## TODO
I have heard many good things about ![pandoc](https://pandoc.org/) and seen
the output from others who coverted RST to MD, with great success. Feel free 
to try it and share your experience.

# Git Pull Command

Git provides a single command to update your local branch with changes from a remote.
`git pull` is this command. Most of the time it does exactly what you want without
any problems, but you should know that `git pull` is really `git fetch` followed
by `git merge`. So when you pull from a remote, you're actually updating the remote
tracking branch (eg. `origin/mybranch`) and then merging that into your local
branch `mybranch`.

It's good to know that this happens under the hood. Some people prefer to do the
`git fetch` and `git merge` operations separately. Most of the time `git pull` will
do what you want and is an acceptable way to update your local branch with changes
from remote.
