# Code documentation lesson

- [Credit and license](https://metos-uio.github.io/sphinx-docs-example/license/)
 
## Locally build this page 
Before filiing a pull-request you should check that the changes introduced by you still make the site build. Therefor, you should always [build and test locally](https://coderefinery.github.io/sphinx-lesson/contributing-to-a-lesson/#build-and-test-locally) before you ask for a pull request. 

## Usage

How to automatically update the documentation locally during development: 
```
$[~/PROJECT_ROOT] sphinx-autobuild content/ _build
```

The static webpage is hosted on Github and automatically build be pushing changes to the `main` branch. The build web-page is displayed as the stuff build on the `gh-pages` branch. This process is automatic. A guide to the principles can be found [here](https://pythonrepo.com/repo/executablebooks-sphinx-autobuild-python-documentation). 
