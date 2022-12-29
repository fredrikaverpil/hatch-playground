# 😶‍🌫️ hatch-playground

Python project boilerplate;

- Hard-pinned production dependencies via `requirements.txt` (and [pip-tools](https://github.com/jazzband/pip-tools)), managed via [dependabot](https://github.com/dependabot).
- Lenient developer/CI dependencies management in `pyproject.toml`.
- Uses the [hatchling](https://github.com/pypa/hatch/tree/master/backend) build system (part of [Hatch](https://github.com/pypa/hatch)).
- Uses version management control via git tags ([hatch-vcs](https://github.com/ofek/hatch-vcs)).

## Quickstart

### Install project locally

In an activated virtual environment, run:

```bash
# install/update project's production and developer dependencies
pip install --upgrade --requirement=requirements.txt --editable='.[dev]'
```

☝️ use this command from time to time to keep your local development environment's dependencies fresh.

### Build project

Use [pypa/build](https://github.com/pypa/build):

```bash
# requires that you ran `pip install ... --editable='[dev]'`
# so that the PEP-517 build frontend is available
# OR for a minimal setup, run: "pip install build" prior to running this command
python -m build
```

Instead of using [pypa/build](https://github.com/pypa/build), pip can be used:

```bash
pip wheel --no-deps --use-pep517 --wheel-dir=dist .
```

### Release

Since this project uses VCS (git) to determine the package version, first make a git tag (following the [PEP-440](https://peps.python.org/pep-0440/) standard), then build the project:

```bash
git tag v0.1.0
python -m build
```

🍑 for continuous deployments, have CI create the tag based on incremental numbering or [conventional commits](https://www.conventionalcommits.org/).

### Publish

I'm using [gh-action-pypi-publish](https://github.com/pypa/gh-action-pypi-publish) in CI, which triggers on pushed tag.

## Maintaining the dependencies

I've opted for hard-pinning all my production dependencies but allowing a more lenient approach for development dependencies.

🍎 If you're building a library or have no need to hard-pin all your production dependencies (maybe you just want to pin the top-level ones), you may want to skip hatch-requirements-txt/pip-tools and just define the dependencies directly in `pyproject.toml`.

### Required dependencies

Pip-tools is used to manage all production dependencies. Edit `requirements.in`, then generate the `requirements.txt` by running the following command:

```bash
pip-compile
```

Dependabot supports pip-tools and can be [set up](https://github.com/fredrikaverpil/hatch-playground/blob/main/.github/dependabot.yml) to prompt for updating these dependencies, one by one.

### Optional dependencies

Different dependency groups are specified in `pyproject.toml` under `[project.optional-dependencies]`.

These can be installed by CI or (optionally) in development. Example, where we make sure we constrain the production dependencies to the pinned ones:

```bash
pip install --upgrade --requirement=requirements.txt --editable='.[tests]'
pip install --upgrade --requirement=requirements.txt --editable='.[docs]'
pip install --upgrade --requirement=requirements.txt --editable='.[lint]'
# and so on...
```

🍌 it's likely a good idea to range-pin some important development dependencies to their major versions.

Update these dependencies manually from time to time, if you pin them in any way. Check what is outdated using pip:

```bash
pip list --outdated
```

CI should fail tests etc if some optional dependency is acting up. Then pin this dependency to a working version and track the issue until it can be unpinned again.

## Background

Why does anyone need this?

I've used [Poetry](https://github.com/python-poetry/poetry) extensively for the last couple of years both for open source and internal/proprietary applications and libraries. There is a considerable amount of churn over time with developer dependencies if you maintain many projects. Especially since I average around 30 developer/CI dependencies per Python repository.

This whole project started as a result of Poetry not allowing you to specify unpinned dependencies, or range-pin dependencies.

Therefore I created [poetry-update](https://github.com/fredrikaverpil/poetry-update), a project which aims to easen the dependabot fatigue for (developer-) dependencies that change often, and where you likely want to always use the latest and greatest. However, this is like a bandaid for a somewhat inflexible dependency management system in my opinion.

Because of this, I set out to see what can be offered by the packaging community outside of the Poetry ecosystem.

Unfortunately, Python does not yet have an accepted PEP specifying a lockfile mechanism to go with `pyproject.toml` that pip would use out of the box. But with Hatchling, the Hatch backend system, it seems I can finally achieve dependency updating zen in the meantime.
