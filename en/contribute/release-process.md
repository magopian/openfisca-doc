# Release process

> This section is for maintainers who want to build and release a Python package of OpenFisca on the [PyPI](https://pypi.python.org/pypi) repository.

Here are the steps to follow to build and release a Python package.
Execute them on each Git repository you want to publish, in that order:

* [OpenFisca-Core](https://github.com/openfisca/openfisca-core)
* [OpenFisca-France](https://github.com/openfisca/openfisca-france)
* [OpenFisca-Parsers](https://github.com/openfisca/openfisca-parsers)
* [OpenFisca-Web-API](https://github.com/openfisca/openfisca-web-api)

See also:

* [PEP 440: public version identifiers](http://legacy.python.org/dev/peps/pep-0440/#public-version-identifiers)
* [distributing packages guide](https://python-packaging-user-guide.readthedocs.org/en/latest/distributing.html)
* [setuptools](https://pythonhosted.org/setuptools/setuptools.html)
* [semver](http://semver.org/)

## Steps to execute

### Tests

Open the [build-status](http://www.openfisca.fr/build-status) page to check that the build status of every project is "passing" (green color).

> Check that there are no pending tests by clicking on the badges.
>
> If there are errors, click on a badge to open the corresponding Travis page.
>
> You can also execute the tests by yourself: do `git pull` in every project on the branch `master`, and run `make test` in each project.

### Internationalization (i18n)

If the project does not use [Python-Babel](http://babel.pocoo.org/) to translate strings, skip this section. Check by looking for the `Babel` entry in `install_requires = ` of `setup.py`.

Extract strings to translate from source code to `.pot` file and update `.po` catalog files:

```bash
python setup.py extract_messages update_catalog
```

Translate `.po` catalog files using [poedit](https://poedit.net/) for example:

```bash
poedit /path/to/i18n/fr/LC_MESSAGES/fr.po
```

Ensure that `Project-Id-Version` in `.pot` and `.po` files are correct.

Commit modified files if needed:

```bash
git commit -am "Update i18n translations"
```

Compile catalog `.po` files:

```bash
python setup.py compile_catalog
```

It should display `(100%) translated`.

### Create the release commit

Determine the next `NEW_RELEASE_NUMBER` by removing the `.dev0` suffix. You can also bump the major number if some commits introduced breaking changes since the last release.

Edit `CHANGELOG.md`:

* fill the changes list
```bash
git log --pretty=format:"* %s" LATEST_VERSION_TAG.. | grep -v "Merge > pull request"
```
* ~~`NEW_RELEASE_NUMBER.dev0 - next release`~~ becomes `NEW_RELEASE_NUMBER`
* delete the line `TODO Fill this changes list while developing`

Edit `setup.py` and check that everything is OK, in particular if requirements have evolved.

```python
setup(
    [...]
    version = 'NEW_RELEASE_NUMBER',
    [...]
    )
```

Commit changes, replacing `NEW_RELEASE_NUMBER`:

```bash
git commit -am "Release NEW_RELEASE_NUMBER"
git tag NEW_RELEASE_NUMBER
git push origin master NEW_RELEASE_NUMBER
```

### Publish on PyPI test instance

Try the release process on the [PyPI test instance](https://wiki.python.org/moin/TestPyPI).

Build and [upload](https://python-packaging-user-guide.readthedocs.org/en/latest/distributing.html#uploading-your-project-to-pypi) the package to the PyPI test instance:

```bash
python setup.py bdist_wheel upload -r https://testpypi.python.org/pypi
```

### Publish on PyPI

Build and upload the package to PyPI:

```bash
python setup.py bdist_wheel upload
```

### Test the package installation

Let's check if the package is installable from PyPI without errors
using [virtualenv](https://virtualenv.pypa.io/en/latest/):

```bash
cd /tmp
virtualenv test-openfisca
cd test-openfisca
source bin/activate
pip install OpenFisca-France
python -m openfisca_france.tests.test_basics
deactivate
```

> Test OpenFisca-Web-API installation if you wish.

### Next steps

Do the same for the remaining repositories to release.

When all the suited repositories are released, you can announce the new release on the website news, Twitter, the mailing list, etc.