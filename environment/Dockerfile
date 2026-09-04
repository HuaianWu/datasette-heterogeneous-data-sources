FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=0337fba234bf574629d56be631468ea060495fa0
RUN git clone https://github.com/simonw/datasette . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# No lockfile upstream: pinned with `uv pip compile (as of 2026-08-10; inputs: pyproject.toml; group dev)`,
# i.e. every release newer than the base commit is excluded, resolved for Linux / CPython 3.12
# (the mars-base default interpreter), so the image matches what upstream CI saw at this commit.
RUN pip install --no-cache-dir \
        "accessible-pygments==0.0.5" \
        "aiofiles==25.1.0" \
        "alabaster==0.7.16" \
        "annotated-types==0.8.0" \
        "anyio==4.14.2" \
        "asgiref==3.12.1" \
        "asyncinject==0.7" \
        "babel==2.18.0" \
        "beautifulsoup4==4.15.0" \
        "black==26.3.1" \
        "blacken-docs==1.20.0" \
        "certifi==2026.7.22" \
        "cffi==2.1.1" \
        "charset-normalizer==3.4.9" \
        "click-default-group==1.2.4" \
        "click==8.4.2" \
        "codespell==2.4.3" \
        "cogapp==3.6.0" \
        "colorama==0.4.6" \
        "cryptography==50.0.0" \
        "docutils==0.21.2" \
        "execnet==2.1.2" \
        "furo==2025.9.25" \
        "h11==0.16.0" \
        "httpcore==1.0.9" \
        "httpx==0.28.1" \
        "hupper==1.12.1" \
        "idna==3.18" \
        "imagesize==2.0.0" \
        "iniconfig==2.3.0" \
        "itsdangerous==2.2.0" \
        "jinja2==3.1.6" \
        "markdown-it-py==3.0.0" \
        "markupsafe==3.0.3" \
        "mdit-py-plugins==0.6.1" \
        "mdurl==0.1.2" \
        "mergedeep==1.3.4" \
        "multipart-form-data-conformance==0.1a0" \
        "mypy-extensions==1.1.0" \
        "myst-parser==4.0.1" \
        "packaging==26.3" \
        "pathspec==1.1.1" \
        "platformdirs==4.11.2" \
        "pluggy==1.6.0" \
        "psutil==7.2.2" \
        "pycparser==3.0" \
        "pydantic-core==2.46.4" \
        "pydantic==2.13.4" \
        "pygments==2.20.0" \
        "pytest-asyncio==1.4.0" \
        "pytest-timeout==2.4.0" \
        "pytest-xdist==3.8.0" \
        "pytest==9.1.1" \
        "python-dateutil==2.9.0.post0" \
        "pytokens==0.4.1" \
        "pyyaml==6.0.3" \
        "requests==2.34.2" \
        "ruamel-yaml==0.19.1" \
        "ruff==0.16.2" \
        "setuptools==84.0.0" \
        "six==1.17.0" \
        "snowballstemmer==3.1.1" \
        "soupsieve==2.9.2" \
        "sphinx-autobuild==2025.8.25" \
        "sphinx-basic-ng==1.0.0b2" \
        "sphinx-copybutton==0.5.2" \
        "sphinx-inline-tabs==2025.12.21.14" \
        "sphinx-markdown-builder==0.6.10" \
        "sphinx==7.4.7" \
        "sphinxcontrib-applehelp==2.0.0" \
        "sphinxcontrib-devhelp==2.0.0" \
        "sphinxcontrib-htmlhelp==2.1.0" \
        "sphinxcontrib-jsmath==1.0.1" \
        "sphinxcontrib-qthelp==2.0.0" \
        "sphinxcontrib-serializinghtml==2.0.0" \
        "sqlite-fts4==1.0.3" \
        "sqlite-utils==4.1.1" \
        "starlette==1.6.0" \
        "tabulate==0.10.0" \
        "trustme==1.2.1" \
        "typing-extensions==4.16.0" \
        "typing-inspection==0.4.3" \
        "urllib3==2.7.0" \
        "uvicorn==0.52.1" \
        "watchfiles==1.2.0" \
        "websockets==17.0.1" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
