| [Home](README.md) ▸ **Creating a Project Page** |
|-----|
# Creating a Project Page

Documentation today is critical the developer, it goes beyond the manual (man)
page we all grew up using. With todays demands and limited time, rich
documentation with links, samples and detailed explanation's is what it takes to
attract users.

In this document we will use tools that can extract documentation from our
Ansible modules into ReStructuredText, generate HTML using Sphinx and format our
site with a Sphinx plugin.

The tools we will use in this tutorial are:
* Sphinx
* ansible-doc-extractor
* sphinx-rtd-theme

NOTE:
This document is written in a progressive manner, where it walks you through the
entire process, step by step. In doing so, you will learn how each command,
tool and configuration build on each other so you can take this lesson and
advance your design.

If you ONLY want to generate documentation and skip the lesson, you can jump to
the [Sphinx Themes](#sphinx-themes) section but ensure you have installed and
configured the utilities.

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
- [Sphinx-versions](#sphinx-versions)


## Virtual Environment
We will use a python virtual environment to install our tools and work from
that includes cloning the repositories. Virtual environments keep your existing
system dependencies from being impacted and lends well to migrating this
process to CI/CD.

NOTE:
It has been noted that Python versions less that 3.8.1 result in errors
extracting and generating docs. I have not tested lesser versions, I wrote this
document based on Python 3.8.2.

I will be using the directory `~/webdocs` to stage this.

```
$ mkdir -p ~/webdocs
$ cd ~/webdocs
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
$ git clone git@github.com:ansible-collections/ibm_zos_core.git
$ cd ibm_zos_core
$ rm -rf docs/*
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

Shpinx created the `source/` directory, add a `modules` directory; this
directory will be used for the generated reStructuredText when
`ansible-doc-extractor` reads the Ansible-doc and extracts into *.rst. This a
vary powerful feature, as it reduces the need to maintain docs in two places.
I highly recommend you invest into documenting the options, examples and return
values, this will pay off in the long run.

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

Add the template path to the configuration `source/conf.py`

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

## Git Page
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

Map the gh-pages repo to gh-pages/ dir and add it to a
![worktree](https://git-scm.com/docs/git-worktree):
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

# Pro-tip

I found it helpful to edit the python code parsing the ReStructuredText with
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

Checkout the branch with your updated RST doc.
```
git checkout master
```

Generate the HTML, recall we don't check-in HTML given we can always generate it
but we do want to store RST with each `master` branch tag for reference and use
it to generate HTML.
```
$ cd docs
$ make html
```

Git ![worktrees](https://git-scm.com/docs/git-worktree) allow you to share
changes between branches, this avoids the need to push and clone and greatly
simplifies the updating of the new generated HTML from your `master` branch to
your `gh-pages` branch.

```
$git worktree add gh-pages gh-pages
```

Copy the generated HTML from the `master` branch to `gh-pages` branch using
rysnc (a - recursive, v -verbose). By now it start to make sense why we are
using Git worktrees and generating HTML from the `master` branch RST.

```
$ rsync -av docs/build/html/ gh-pages/
```

With Git work trees you don't checkout the branch, you `cd` to the branch, this
is handled by Git using hard links in their implementation, without this you
would not have been able to `rsync` locally between two branches.

Checkout `gh-pages` branch, add, commit and push your updated HTML.

```
$ cd gh-pages

# At this point you may want to validate your HTML and `open index.html` and
# browse the changes ensure they copied correctly.

$ rm -rf .DS_Store
$ git add .
$ git status
$ git commit -m "Updated documentation version"
$ git push
```

Given you probably don't update HTML to often, you may not have another use for
Git worktrees.

```
$ cd master
$ git worktree remove gh-pages
```
# Research
Things I tried, things that did not work, and items for research.

I tried to use this convert MD to RST with a conversion utility and it did not
correctly generate the RST files, particularly URLs.
```
$ pip install mdToRst
$ mdToRst README.md >README.rst
```

Not sure why I installed
![sphinx-jinja](https://pypi.org/project/sphinx-jinja/), but don't have any
comments.
```
pip install sphinx-jinja
```

I have heard many good things about ![pandoc](https://pandoc.org/) and seen
the output from others who converted RST to MD, with great success. Feel free
to try it and share your experience.

# Sphinx-versions

Its not uncommon that you encounter the need to host more than one version of
documentation on your self-hosted site (gh-pages). For example, you may have
released `v1.0.0` and `v1.1.0-beta.1`, so how do you host two versions of
documentation into one site? Allow users to select between versions? Display
a banner when users are looking at an older version of doc? Luckily there
is a great Sphinx extension called
![sphinx-versions](https://github.com/Smile-SA/sphinx-versions) and
![documentation](https://sphinx-versions.readthedocs.io/en/latest/index.html#).

In my testing, I had run into errors using the latest
![sphinx-versions 1.1.3](https://pypi.org/project/sphinx-versions/1.1.3/) that
after spending hours could not resolve and decided to try out other versions as
that seemed to be what others were having to do. I settled on
![spinx-versions 1.0.0](https://pypi.org/project/sphinx-versions/1.0.0/) for now
and in time will try to solve the errors I encountered in newer versions.

For reference my error:
```
writing output... [  5%] community_guides
Theme error:
An error happened in rendering the page community_guides.
Reason: TypeError('expected str, bytes or os.PathLike object, not NoneType')
Process Process-2:
Traceback (most recent call last):
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/multiprocessing/process.py", line 315, in _bootstrap
    self.run()
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/multiprocessing/process.py", line 108, in run
    self._target(*self._args, **self._kwargs)
  File "/Users/ddimatos/git/python-venv-ansible-zos/venv/lib/python3.8/site-packages/sphinxcontrib/versioning/sphinx_.py", line 213, in _build
    raise SphinxError
sphinx.errors.SphinxError
=> sphinx-build failed for branch/tag: master
```
## Install
I wanted to try out `pipenv` so my commands will include its usage but i will
also share the `pip` commands.

Use `pip` to install `pipenv` and update your PIP if needed:
```
 $ pip3 install -U pip
 $ pip install --user -U pipenv
 ```

Install `sphinx-versions` with `pipenv`:
```
pipenv install sphinx-versions
```

Or, install `sphinx-versions` with `pip`:
```
pip install sphinx-versions==1.0.0
```

## Generating documentation
Building off the prior Sphinx tutorial where we created HTML from
RestructuredText, I will assume you have content for more than one release.
The `sphinx-versioning` command will call `sphinx` such that it not only
generates HTML, it also creates the selectable versions of doc. Its
best you start off by running `sphinx-versioning` from your repository root.
Running the below command will generate doc for all your branches and Git Tags
which usually  not desired behavior.

```
sphinx-versioning -l docs/source/conf.py build  docs/source/ docs/build/html
```

You now view what `sphinx-versioning` has generated with command:
```
open docs/_build/html/index.html
```

An example of how it looks when all your branches and tags are versioned:

<img width="544" alt="image" src="https://user-images.githubusercontent.com/25803172/82095423-d5e3a600-96b3-11ea-94aa-6a749f8bc77d.png">

<img width="593" alt="image" src="https://user-images.githubusercontent.com/25803172/82094690-691bdc00-96b2-11ea-8e3d-e6c435d2e836.png">

## Configure
At this point you probably are noticing you have documentation generated for
all your Git tags and branches which is usually not desirable behavior. You can
control what `sphinx-versioning` branches, tags will be versioned as well as
the features you would like added such as a banner that warns users they are
viewing an older version of doc by reviewing the `sphinx-versioning`
![configuration reference](https://sphinx-versions.readthedocs.io/en/latest/settings.html).

I have configured `sphinx-versioning` to create documentation only for Git tags
because we only tag releases in the `master` branch and other other branches
could possibly have older documentation and depending on when they branch off of
the `dev` branch, thus we don't want doc for those. I have also configured that
our documentation display links in **semantic** ordering, display a banner that
a user is on an older version by setting that option also to
our **semantic** version.

Below are snippets of the configuration and some added documentation and
explanations for my choices. This is the same configuration file we used to
configure Sphinx and added to it, and you can see it
![here](https://github.com/ansible-collections/ibm_zos_core/blob/dev/docs/source/conf.py)

```
# Choosing to not generate documentation on any branch and rely solely on
# Github tags. Branches are whitelisted with option 'scv_whitelist_branches'.
# In other words, filter out any branches that don't match the pattern.
scv_whitelist_branches = (' ',)

# Override root-ref to be the tag with the highest version number. If no tags
# have docs then this option is ignored and --root-ref is used. Since we
# whitelist the master branch, we need to set a root_ref.
# See also 'scv_root_ref
scv_greatest_tag = True

# White list which Git tags documentation will be generated and linked into the
# version selection box. This is currently a manual selection, until more
# versions are released, there are no regular expressions used.
scv_whitelist_tags = ('v1.0.0', 'v1.1.0-beta1')

# Sort versions by one or more values. Valid values are semver, alpha, and time.
# Semantic is referred to as 'semver', this would ensure our latest VRM is
# the first in the list of documentation links.
scv_sort = ('semver',)

# Show a warning banner. Enables the Banner Message feature. Further info:
# https://sphinx-versions.readthedocs.io/en/latest/banner.html#banner
scv_show_banner = True

# Override banner-main-ref to be the tag with the highest version number. If no
# tags have docs then this option is ignored and --banner-main-ref is used.
# The greatest tag is desirable behavior for this site.
scv_banner_greatest_tag = True

```

Now that you have configured `sphinx-versioning` to your repository desired
options, if you repeat the command from earlier
`sphinx-versioning -l docs/source/conf.py build  docs/source/ docs/build/html`
you should see documentation generated and linked per your configuration.

## Automating

In the prior Sphinx tutorial I covered how to use a Make file to automate some
of the repeated steps; I have done the same for `sphinx-versioning`. You can
view the Makefile
![here](https://github.com/ansible-collections/ibm_zos_core/blob/dev/docs/Makefile).

This Makefile is configured for our repositories specific needs such as
generating doc for non-python languages but there are snippets you can use
in your own Makefile.

When you run `sphinx-versioning` from a Makefile its not likely running from
your repository root so you need to provide absolute paths, as I was not able
to have relative links work. To to this i needed to know the where the
Makefile was running with this addition to the Makefile:

```
ROOT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
```

Now I could use `ROOT_DIR` to build the `sphinx-versioning` command.

```
html-all:
	@sphinx-versioning -l "$(ROOT_DIR)"/source/conf.py build  "$(ROOT_DIR)"/source/ "$(ROOT_DIR)"/build/html
	@echo "Completed HTML generation for git repository branches and/or tags, run 'make view-html'"
```

You no longer need to run `make html` to generate doc, you can run
`make html-all` to generate doc and version it. `sphinx-versioning` will handle
calling and passing arguments to `sphinx` to generate doc.

After this you can view the html once again with:
```
open build/html/index.html
```

If you have chosen to version only Git tags, that should look like:
<img width="599" alt="image" src="https://user-images.githubusercontent.com/25803172/82095477-ed229380-96b3-11ea-9ebb-8e60495bb531.png">

Thats all we need to do to support our own hosted documentation with versioning.
