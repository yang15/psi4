.. #
.. # @BEGIN LICENSE
.. #
.. # Psi4: an open-source quantum chemistry software package
.. #
.. # Copyright (c) 2007-2019 The Psi4 Developers.
.. #
.. # The copyrights for code used from other parties are included in
.. # the corresponding files.
.. #
.. # This file is part of Psi4.
.. #
.. # Psi4 is free software; you can redistribute it and/or modify
.. # it under the terms of the GNU Lesser General Public License as published by
.. # the Free Software Foundation, version 3.
.. #
.. # Psi4 is distributed in the hope that it will be useful,
.. # but WITHOUT ANY WARRANTY; without even the implied warranty of
.. # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
.. # GNU Lesser General Public License for more details.
.. #
.. # You should have received a copy of the GNU Lesser General Public License along
.. # with Psi4; if not, write to the Free Software Foundation, Inc.,
.. # 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
.. #
.. # @END LICENSE
.. #

.. include:: autodoc_abbr_options_c.rst


Release Procedures
==================


.. _`faq:annualprocedure`:

Annual
------

* `Update copyright year`_


.. _`faq:prereleaseprocedure`:

Pre-Release (e.g., ``v1.3rc1``)
-------------------------------

* `Update samples`_
* `Collect new authors`_
* `Anticipate next release`_
* `Build Conda ecosystem stack`_
* `Tag (pre)release`_
* `Build Conda Psi4 stack at specific commit`_
* `Build Psi4conda set`_
* `Generate download page for psicode.org`_
* `Reset psi4meta for nightly operation`_


.. _`faq:releaseprocedure`:

Release (e.g., ``v1.3``)
------------------------

* `Do final pass before release tag`_
* `Tag (pre)release`_
* `Initialize release branch`_
* `Build Conda Psi4 stack at specific commit`_
* `Publish to main conda label`_
* `Build Psi4conda set`_
* `Generate download page for psicode.org`_
* `Publish GitHub release`_
* `Publish psicode release`_
* `Finalize release`_
* `Reset psi4meta for nightly operation`_


.. _`faq:postreleaseprocedure`:

Post-Release (e.g., ``v1.3.1``)
-------------------------------



Update copyright year
---------------------

* ``cd ~/path/to/psi4``
* Primary target is licenses

  - ``grep -rl "(c) 2007-2017" * | xargs sed -i '' "s/(c) 2007-2017/(c) 2007-2018/g"``
  - On Linux, drop the ``''`` in above command
  - Need to do ``psi4/`` and ``docs/`` dirs

* Also, license in these files

  - ``tests/runtest.py``
  - ``README.md``
  - ``tests/psitest.pl``

* Also, in content of https://github.com/psi4/psi4/blob/master/doc/sphinxman/source/conf.py.in#L118


Update samples
--------------

* Run ``make sphinxman`` at least once by hand
* Check in resulting ``psifiles.py`` and all the updated and new ``samples/`` files and dirs
* Make a lone PR and warn reviewers not to read it, since autogenerated


Collect new authors
-------------------

* Survey contributions to current Milestone. Add new contribs and PR lists to release notes GH issue
* Figure out any new "Additional Contributors" authors since last release
* Edit ``psi4/header.py`` accordingly, make PR
* Get permission of new authors and their particulars for ``codemeta.json``
* Invite any contributors with at least 3 PRs to join GitHub Organization


Anticipate next release
-----------------------

* Bump version in ``codemeta.json``, https://github.com/psi4/psi4/blob/master/codemeta.json#L9
* Add to branch list in ``.travis.yml``, https://github.com/psi4/psi4/blob/master/.travis.yml#L179


Build Conda ecosystem stack
---------------------------

By "ecosystem stack", mean packages that are upstream, downstream, required, and optional for a fully featured Psi4 build and which we can't get from "defaults" or "conda-forge" channels.

* Main directions are in [cbcy](https://github.com/psi4/psi4meta/blob/master/conda-recipes/conda_build_config.yaml#L90-L103) and [poodle](https://github.com/psi4/psi4meta/blob/master/psinet-nightly/kitandkapoodle.py#L357-L411)
* A couple weeks before the first "rc" is planned, start going through L/LT in poodle, checking with upstream to see if new versions have been released. If good changes present, rebuild the packages, changing the version numbers in the respective recipes
* When L/LT all built and passed, edit the individual package version numbers in cbcy and increment to a new ``ltrtver`` with updated version numbers and/or build numbers (only if code changes)
* Build L/PSI4. If any trouble, edit psi4 code. Iterate until builds and passes. This stage is the only full ctest & pytest on Psi4+upstream
* Build L/RT-MP. If any trouble, edit code in L/RT and rebuild those package(s). Iterate until builds and passes. This stage is the only full ctest & pytest of Psi4+downstream
* Build L/DEV. If any trouble, edit psi4 build system, plugin system, or OpenMP setup. Iterate until builds and passes
* Build L/DOCS. If any trouble, edit the docs or the tests. Iterate until builds and passes
* Results of last should upload to psicode.org (docs) and codecov.io (coverage)
* Changes to targets' "source" and "version" in individual recipes should be edited in psi4 ``external/`` CMakeLists.txt files
* Once everything's working on Linux, repeat on Mac
* At this point, ready to fine tune builds of "Psi4 stack"


Do final pass before release tag
--------------------------------

* Check that ``external/`` repos and commits have been updated to match conda recipes sources. Also check versions with ``conda_build_config.yaml``
* Check ``docs/sphinxman/source/introduction.rst`` for any compiler and Python minimum requirements to edit.


Tag (pre)release
----------------

* Thorough version bump directions at master http://psicode.org/psi4manual/master/manage_git.html#how-to-bump-a-version
* Below is tl;dr

  ::

    # be on clean master up-to-date with upstream in both commits and tags
    # * mind which version strings get "v" and which don't
    # * if not fork, replace "upstream" with "origin"

    >>> vi psi4/metadata.py
    >>> git diff
    diff --git a/psi4/metadata.py b/psi4/metadata.py
    ...
    -__version__ = '1.3rc1'
    -__version_long = '1.3rc1+5a7522a'
    -__version_upcoming_annotated_v_tag = '1.3rc2'
    +__version__ = '1.3rc2'
    +__version_long = '1.3rc2+zzzzzzz'
    +__version_upcoming_annotated_v_tag = '1.3rc3'

    >>> git add psi4/metadata.py
    >>> git commit -m "v1.3rc2"
    [master bc8d7f5] v1.3rc2

    >>> git log --oneline | head -1
    bc8d7f5 v1.3rc2
    >>> git tag -a v1.3rc2 bc8d7f5 -m "v1.3rc2"

    >>> vi psi4/metadata.py
    >>> git diff
    diff --git a/psi4/metadata.py b/psi4/metadata.py
    ...
    -__version_long = '1.3rc2+zzzzzzz'
    +__version_long = '1.3rc2+bc8d7f5'

    >>> git add psi4/metadata.py
    >>> git commit -m "Records tag for v1.3rc2"
    [master 16dbd3e] Records tag for v1.3rc2

    # goto GH:psi4/psi4 > Settings > Branches > master > Edit
    #      https://github.com/psi4/psi4/settings/branch_protection_rules/424295
    # uncheck "Include administrators" and Save changes

    >>> git push upstream master
    >>> git push upstream v1.3rc2

    # re-engage "Include administrators" protections


Initialize release branch
-------------------------

* follow tagging procedure
* before reengaing the "include admin" button, push a branch at the tag commit (not the records commit)

  ::

    >>> git log --online | head -2
    45315cb Records tag for v1.3
    20e5c7e v1.3

    >>> git checkout 20e5c7e
    >>> git checkout -b 1.3.x
    Switched to a new branch '1.3.x'
    >>> git push upstream 1.3.x

* set up new branch as protected branch through GH psi4 org Settings


Build Conda Psi4 stack at specific commit
-----------------------------------------

By "Psi4 stack", mean packages ``psi4``, ``psi4-rt``, ``psi4-dev``, ``psi4-docs``.
Other packages, the "ecosystem stack" (e.g., ``libint``, ``v2rdm_casscf``) should be already built.

* Particularly before release (not prerelease), consider max pinnings on dependencies, particularly any fast-moving deps (e.g., qcel) and whether they need version space to grow compatibly and grow incompatibly.
* Nightly conda-builds work from ``master`` psi4. Instead, switch ``source/git_tag`` from ``master`` to tag (e.g., ``v1.3rc1``) in:

  - psi4-multiout on Linux & Mac, https://github.com/psi4/psi4meta/blob/master/conda-recipes/psi4-multiout/meta.yaml#L10
  - psi4-docs on Linux, https://github.com/psi4/psi4meta/blob/master/conda-recipes/psi4-docs/meta.yaml#L10 on L

* For releses (not prereleases), in ``conda_build_config.yaml``, edit ``ltrtver`` to a new non-dev label (probably a ditto) matching the release (e.g., "1.3")
* Set ``kitandkapoodle.py`` to the normal ``***`` stack. Should be (``psi4``, ``psi4-rt``, ``psi4-dev``) * python_versions for Linux & Mac. Also single ``psi4-docs``     from Linux
* Run ``kitandkapoodle.py`` and allow stack to upload to anaconda.org to ``psi4/label/dev``. Poodle emits with ``--label dev`` so will go to the subchannel. May need to delete packages to clear out space on anaconda.org


Publish to main conda label
---------------------------

* Go through each active conda package off https://anaconda.org/psi4/repo

  - Find the most recent build set (Linux/Mac, active py versions) that psi/psi-rt/psi-dev is using
  - _add_ (not replace) the ``main`` label.

* This makes a ``conda install psi4 -c psi4`` get everything psi4 needs. For the moment ``conda install psi4 -c psi4/label/dev`` will get the same set, until package psi4-1.4a1.dev1 gets uploaded. May help to check versions and build versions against ltrtver in ``conda_build_config.yaml``.
* This step is manual, so takes a while. (It gets checked when Psi4conda installers are built b/c that pulls from "main", not "dev")


Build Psi4conda set
-------------------

Installers are build using the project ``constructor`` and build binary bash scripts, one per OS per Python version. In analogy to Miniconda, they're called Psi4Conda. They can be built anywhere (Mac can be built on Linux) and get served from vergil (cdsgroup webserver).

* Need a conda env with ``constructor`` and ``cookiecutter``. This env presently accessed through ``conda activate cookie``.
* Enter "constructor-cutter-unified" in the psi4meta repo. There's a good README there, https://github.com/psi4/psi4meta/blob/master/conda-recipes/constructor-cutter-unified/README.md
* Edit ``cookiecutter/cookiecutter.json`` for control

  - Edit which python versions, if necessary
  - Edit ``release`` field
  - Edit ``hash`` field. This is the 7-char hash that's on every psi4 conda pkg as part of version
  - Edit ``ltrtver`` field. This matches the current setting in ``conda_build_config.yaml``
  - For prereleases, ``"channel_tag": "/label/dev"``, while for releases, it should be the empty string
  - Leave this file set to a "rc" with Git, as that has more details

* For releases (not prereleases), copy cookiecutter.json to cookiecutter.json-vXXX
* Edit ``cookiecutter/{{.../construct.yaml`` for templating. This is rarely needed
* If it's been a while or you need the space, clear out ``~/.conda/constructor``, where the downloaded packages are cached
* Note that installers get written to ``build/`` and this gets regenerated each time. Clear out between runs.
* ``python run.py``
* Watch out for ``py_`` in buildstring as this means a noarch package has been pulled. It must be eliminated. Constructors can't handle "noarch" packages and will fail at runtime. If see a "noarch" package, must find the recipe and rebuild for all OS & Python combinations. Then run constructor again.
* If fetching times out, may have to run run.py several times. Clear out build/ in between. It's the fetching that takes a long time, not constucting
* In the end, should have several installers ::

  >>> lh build/psi4conda-1.3-py3.*/*64.sh
  -rwxr-xr-x. 516M Feb 28 20:30 build/psi4conda-1.3-py3.6-linux-64/psi4conda-1.3-py36-Linux-x86_64.sh
  -rwxr-xr-x. 299M Feb 28 20:31 build/psi4conda-1.3-py3.6-osx-64/psi4conda-1.3-py36-MacOSX-x86_64.sh
  -rwxr-xr-x. 518M Feb 28 20:30 build/psi4conda-1.3-py3.7-linux-64/psi4conda-1.3-py37-Linux-x86_64.sh
  -rwxr-xr-x. 299M Feb 28 20:31 build/psi4conda-1.3-py3.7-osx-64/psi4conda-1.3-py37-MacOSX-x86_64.sh


* Upload installer files to vergil, ``scp -r build/Psi4*/Psi4*sh root@vergil.chemistry.gatech.edu:/var/www/html/psicode-download/``
* Log in to vergil root and make WindowsWSL symlinks


Generate download page for psicode.org
--------------------------------------

* Be in repo psicode-hugo-website
* Copy and edit new file akin to https://github.com/psi4/psicode-hugo-website/blob/master/content/installs/v13rc2.md. Note the edition string ``v13rc2`` in frontmatter for this and future filenames
* Copy and edit new file akin to https://github.com/psi4/psicode-hugo-website/blob/master/data/installs/v13rc2.yaml for menu and notes content
* Enter ``scripts/`` dir and edit primarily https://github.com/psi4/psicode-hugo-website/blob/master/scripts/install-generator.py#L9 but also any other arrays or messages that should change.
* Run the  ``install-generator.py`` in place. It will dump new files into ``data/installs/`` _subdirs_. Be sure to ``git add`` them.
* Installer page is now ready.
* Shift "latest" alias in frontmatter from whichever page is currently active to the new page. This makes sure "Downloads" on the navigation bar points to new page.
* Conscientiously, on should test

  - installer downloads in Mac and Linux. And actually installing them and ``psi4 --test`` them.
  - that download button and ``curl`` downloading register on the download counters on vergil

* Commit the new files, PR, and deploy psicode site
* Petition on Slack for testers


Publish GitHub release
----------------------

* On GH site "Draft a New Release" with newly minted tag
* Fill in frontmatter style and links from previous GH release
* Fill in RN from hopefully existing RN issue
* Fill in RN by going through the top posts from all PRs from this milestone
* "publish" release. This establishes release date for GH api
* Close the RN issue.
* Close the milestone (should be 100% complete).


Publish psicode release
-----------------------

* Copy a recent release page like https://github.com/psi4/psicode-hugo-website/blob/master/content/posts/v1p2.md
* Edit its filename, title, date, image, and links
* Execute https://api.github.com/repos/psi4/psi4/releases/latest and note the ``id`` field value
* Use the ``id`` value in the shortcode call at the bottom


Finalize release
----------------

* Make new PR with
  * edits to main ``README.md`` badges, python versions, etc.
  * edits to ``CMakeLists.txt`` ``find_package(PythonLibsNew 3.6 REQUIRED)``
* On godaddy, grab the exact tag 1.3 manual before it gets overwritten and preserve it: ``cp -pR master 1.3``
* Tweet about release


Reset psi4meta for nightly operation
------------------------------------

On both Linux and Mac:

* After release (not prerelease), in ``conda_build_config.yaml``, edit ``ltrtver`` to a new "release.dev" label
* Edit ``source/git_tag`` back to ``master`` for psi4-multiout, psi4-docs
* Edit build string back to ``0`` if psi4-multiout needed multiple passes
* Edit kitandkapoodle.py back to ``***`` stack
* Check in all release, construct, recipe changes on Linux and Mac. Synchronize both to GH psi4meta
* Edit crontab back to 2am "norm". Comment out "anom"

