# Contribution 

This documentation is not meant to be static and is a collaborative effort of the CTSM-Norway team. If you spot anything wrong or would like to add information to the documentation, please send us a pull request or file an issue in our [repository](https://github.com/MetOs-UiO/CTSM-Norway-Documentation).


```{prereq} What do I need to contribute?
1. Basic understanding of Git.

2. You need to have [Sphinx](http://www.sphinx-doc.org) installed (as
  part of your Python environment installation).

3. You need a [GitHub](https://github.com) account.
```

## Locally build this page 
Before filling a pull-request you should check that the changes introduced by you still make the site build. Therefor, you should always [build and test locally](https://coderefinery.github.io/sphinx-lesson/contributing-to-a-lesson/#build-and-test-locally) before you ask for a pull request. 

### Usage

How to automatically update the documentation locally during development: 
```
$[~/PROJECT_ROOT] sphinx-autobuild content/ _build
```

The static webpage is hosted on Github and automatically builded with GitHub-Actions by pushing changes from the `main` to the `gh-pages` branch. The built web-page is displayed as the stuff build on the `gh-pages` branch. This process is automatic. A guide to the principles can be found [here](https://pythonrepo.com/repo/executablebooks-sphinx-autobuild-python-documentation). 
