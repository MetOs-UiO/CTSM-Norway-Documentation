# Code documentation lesson

- [Credit and license](https://metos-uio.github.io/sphinx-docs-example/license/)
 
## The recommended workflow for development
1. Raise an issue, add tags and assign yourself to make others aware of your plan.
2. Fork the repository to your Github account and fetch upstream to receive the newest commits.
3. Perform the development in a newly-created branch (other than `main`) in your repository. 
4. Build the page in the command line to test the introduced modifications by following the instructions below.

### Locally build this page 
Before filling a pull-request you should check that the changes introduced by you still make the site build. Therefore, you should always [build and test locally](https://coderefinery.github.io/sphinx-lesson/contributing-to-a-lesson/#build-and-test-locally) before you ask for a pull request. 

To automatically update the documentation locally during development: 
```
$[~/PROJECT_ROOT] sphinx-autobuild content/ _build
```

The static webpage is hosted on Github and automatically built by pushing changes to the `main` branch. The built webpage is displayed when selecting the `gh-pages` branch. This process is automatic. A guide to the principles can be found [here](https://pythonrepo.com/repo/executablebooks-sphinx-autobuild-python-documentation). 
