---
layout: default
title: Developer Guide for updating python SDK docs
parent: Feathr Developer Guides
---
# Developer Guide for updating python SDK docs

## Install Dependencis

`pip install sphinx &&  pip install sphinx_rtd_theme`

Since `sphinx` need to run the source code to generate the documentation, make sure you have installed the all the necessary dependencies needed by the project(see setup.py files).

## How Readthedocs.com Works
* Readthedocs need to run the Python code to generate the docs. So your dependencies need to exist on the machine that generates these docs.
* Readthedocs can be configured via `.readthedocs.yml` under project root(not your Python library root). If you don't 
  need extra configurations, you don't need `.readthedocs.yml`.
* Readthedocs will generate documentations based on your `.rst` file. `.rst` file can be automatically generated or manually modified(see later sections).
* Readthedocs doesn't support complex dependencies, like Pypi packages with C modules. To address this, Readthedocs provides those common
  dependencies in their server but we need to enable it by setting `system_packages: true`
* Dependencies can be specified by docs/requirements.txt file.
* You don't need to commit _build files. Those files will be generated by the Readthedocs.com servers.

### How to Specify Dependencies
Readthedocs provides two ways of specifying depencies.
The first one is via the docs/requirements.txt file. This one will need to duplicate some of the information from the setup.py file. The benefit is we can pick only the dependencies that are needed by the execution of Readthedocs.
```
python:
  install:
    - requirements: feathr_project/docs/requirements.txt
```

The second one is using setup.py. This one avoids the duplication but have to install all dependencies. Readthedocs have poor support on importing complex dependencies, like Pandas can't be imported. So this really doesn't work for us.
```
python:
  install:
    - method: setuptools
      path: feathr_project/
```
So using requirements.txt is used.

In the future, we should simplify the dependencies for user facing APIs. But it's hard to do the same for developer-facing APIs. We still count on Readthedocs to sipmlify and address the dependency importing issues.

### How to Edit Contents
You can edit the rst files to modify the structure and contents of the docs page.

## Build the documentation html files

Then rebuild the html files:

`make clean && make html`

You will see new html files generated under `_build/html/` directory and you can view `_build/html/index.html` in your browser locally.

## Re-build the documentation html files
If you need to re-build the `.rst` files, run the following command to update them:

In docs directory:

`sphinx-apidoc -f -o . ../feathr ../*setup*`

(excluding setup.py files, and some other demo files, test files.)

## Exposing the right namespace in Pydocs
Currently, the code is structured as this:
```
feathr_project/feathr
├── __init__.py
├── definition
│   ├── _materialization_utils.py
│   ├── aggregation.py
├── protobuf
│   ├── __init__.py
│   └── featureValue_pb2.py
```
When the end users need to import Feathr modules, for example aggegations, it should be straightforward for them to do so. Currently they should use:

```python
from feathr import Aggregation
```
rather than 

```python
from feathr.definition import Aggregation
```
And this namespace should also be set correctly in the pydocs. 

According to [this answer in StackOverflow](https://stackoverflow.com/questions/15115514/how-do-i-document-classes-without-the-module-name/31594545#31594545), we are doing the following:

1. Add an `__all__` section in `__init__.py` (see code [here](../../feathr_project/feathr/__init__.py)). Every components that are included in the `__all__` section is exposed to end users. Others are not exposed in the pydocs.
2. In the [rst file](../../feathr_project/docs/feathr.rst), just use a single module:
```
.. automodule:: feathr
    :members:
    :undoc-members:
    :show-inheritance:
```

So that only this module is accessbile for end users.

## Upload to Readthedocs.com
* Login to https://readthedocs.org/dashboard/
* Click `Import a Project`
* Click `Import Manually` on the right side
* Fill in `name`, `Repository URL`(the url to feathr main), and `Default branch`(main or the branch you want to test).
* Click `Next`

### Test
* After you have imported your own branch, you can click `Build version` to test the build result of your latest code on the branch.
* You can click on each pannels to see the command message and warnings.
* After the build is successful, it will show
  the docs page(like https://xxx.readthedocs.io/en/latest/feathr.html). But they have a site cache issue. You have to refresh the site then you can see your new result.
* Sometimes the python docs are not correctly formatted and you will see the build is successful, but you won't see any docs (just blank pages). You will see error messages like below, **though the build is successful**. Pleae make sure you fix those errors.

```
/home/docs/checkouts/readthedocs.org/user_builds/feathr-xiaoyzhu/checkouts/latest/feathr_project/feathr/client.py:docstring of feathr.client.FeathrClient.register_features:5: ERROR: Unexpected indentation.
```

### Debug and Known Issues
* `No module named xyz`: Readthedocs need to run the code to generated the docs. So if your dependency is not specified
in the docs/requirements.txt, it will fail on this. To fix it, specify the dependency in requirements.txt.
