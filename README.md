# rk-docs
This is the repository which holds the site of https://rkdocs.mofcloud.com

## How to run?
This site uses mkdocs as framework. User needs to install mkdocs on local development environment.

### Install dependencies
```bash
pip install mkdocs
pip install mkdocs-material
pip install mkdocs-glightbox
pip install mkdocs-static-i18n
pip install mkdocs-video
```

### Run on localhost
Run bellow command and access website via http://127.0.0.1:8000/

```bash
mkdocs serve

...
INFO     -  [05:24:53] Serving on http://127.0.0.1:8000/
...
```

### Add content
Contents of markdown files can be added into docs/ folder. Meanwhile, user needs to add navigation and nav translation in mkdocs.yml file as needed.

Please refer to examples in code.

## Contributing
We encourage and support an active, healthy community of contributors &mdash;
including you! Details are in the [contribution guide](CONTRIBUTING.md) and
the [code of conduct](CODE_OF_CONDUCT.md). The rk maintainers keep an eye on
issues and pull requests, but you can also report any negative conduct to
lark@pointgoal.io.

Released under the [Apache 2.0 License](LICENSE).
