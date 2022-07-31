# hatch-playground

Project maintenance with [hatch's hatchling](https://github.com/pypa/hatch), [hatch-vcs](https://github.com/ofek/hatch-vcs), [hatch-requirements-txt](https://github.com/repo-helper/hatch-requirements-txt); a boilerplate.

Uses the following PEPs:
- [PEP-621](https://peps.python.org/pep-0621/) - Storing project metadata in pyproject.toml
- [PEP-517](https://peps.python.org/pep-0517/) - A build-system independent format for source
- [PEP-508](https://peps.python.org/pep-0508/) - Dependency specification for Python Software Packages
- [PEP-440](https://peps.python.org/pep-0440/) - Version Identification and Dependency Specification
...and probably others.

## Quickstart

### Install project locally

```bash
# just install the project with the production dependencies
# as editable (developer mode)
pip install -e .
```

```bash
# install with all developer dependencies, as editable (developer mode)
# and constrained to the pinned production dependencies
pip install --upgrade -e '.[dev]' -c requirements.txt
```

☝️ use this command from time to time to keep your local development environment's dependencies fresh.

### Build project

```bash
# install PEP-517 front-end tools
pip install build

# build the project's distributables
python -m build
```

### Release

Since this project uses VCS (git) to determine the package version, first make a git tag, then build the project:

```bash
git tag v0.1.0
python -m build
```

The git tag can be bumped accordingly by CI on merge to the main branch, using conventional commits.

### Publish

Use any tool of your liking. I'm using [Twine](https://github.com/pypa/twine/) (part of the developer dependencies). This is likely something you want to be part of your CI, triggered on a new git tag/release.

## Maintaining the dependencies

I've opted for hard-pinning all my production dependencies but allowing a more lenient approach for development dependencies.

🍎 If you're building a library or have no need to hard-pin all your production dependencies (maybe you just want to pin the top-level ones), you may want to skip hatch-requirements-txt/pip-tools and just define the dependencies directly in `pyproject.toml`.

### Required dependencies

Pip-tools is used to manage all production dependencies. Edit `requirements.in`, then generate the `requirements.txt`:

```bash
pip-compile --no-header --no-annotate
```

Dependabot supports pip-tools and can be set up to prompt for updating these dependencies, one by one.

### Optional dependencies

Different dependency groups are specified in `pyproject.toml` under `[project.optional-dependencies]`.

These can be installed by CI or (optionally) in development. Example, where we make sure we constrain the production dependencies to the pinned ones:

```bash
pip install '.[docs]' -c requirements.txt
```

🍌 it's likely a good idea to pin some important development dependencies to their major versions.

Update these dependencies manually from time to time, if you pin them in any way. Check what is outdated using pip:

```bash
pip list --outdated
```

CI should fail tests etc if some optional dependency is acting up. Then pin this dependency to a working version and track the issue until it can be unpinned again.

## Background

Why does anyone need this?

I've used [Poetry](https://github.com/python-poetry/poetry) extensively for the last couple of years both for open source and internal/proprietary applications and libraries. There is a considerable amount of churn over time with deveoloper dependencies if you maintain many projects. Especially since I average around 30 developer/CI dependencies per Python repository.

This whole project started as a result of Poetry not allowing you to specify unpinned dependencies, or range-pin dependencies.

Therefore I created [poetry-update](https://github.com/fredrikaverpil/poetry-update), a project which aims to easen the dependabot fatigue for (developer-) dependencies that change often, and where you likely want to always use the latest and greatest. However, this is like a bandaid for an inflexible dependency management system in my opinion.

Because of this, I set out to see what can be offered by the packaging community outside of the Poetry ecosystem.

Unfortunately, Python does not yet have an accepted PEP specifying a lockfile mechanism to go with `pyproject.toml` that pip would use out of the box. But with Hatchling, the Hatch backend system, it seems I can finally achieve dependency updating zen in the meantime.
