# CI/CD для статического сайта в SourceCraft

Сайт: [https://sourcecraft.dev/pnflvnn-gmail-com/portfolio](https://sourcecraft.dev/pnflvnn-gmail-com/portfolio)  
Репозиторий SourseCraft: [https://pnflvnn-gmail-com.sourcecraft.site/portfolio](https://pnflvnn-gmail-com.sourcecraft.site/portfolio)

---

## Цель

Научиться настраивать автоматическую сборку и публикацию статического сайта с помощью CI/CD на платформе SourceCraft

---
## Задачи

- Создать репозиторий на SourceCraft

- Настроить автоматическую сборку сайта MkDocs

- Опубликовать сайт на SourceCraft

---

## Код

<p align="center">ci.yaml</p>

```python
on:
  push:
    workflows: build-site
    filter:
      branches: main

workflows:
  build-site:
    tasks:
      - name: build-and-publish-site
        cubes:
          - name: build-mkdocs-site
            image: docker.io/library/python:3.13-slim
            script:
              - python -m pip install --upgrade pip
              - "if [ -f requirements.txt ]; then pip install -r requirements.txt; fi"
              - cd source && mkdocs build -d ../docs
              - echo "Сайт собран в папке ./docs"
              - ls -la ./docs

          - name: publish-to-release-branch
            script:
              - git checkout -b release
              - git add ./docs
              - "git commit -m \"feat: автоматическое обновление сайта\""
              - "git push origin release -f"
```

<p align="center">sites.yaml</p>

```python
site:
    root: docs
    ref: release
```
<p align="center">deploy.yaml</p>

```python
on:
  push:
    branches: [ main ]

permissions:
  contents: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.13'

      - name: Install dependencies
        run: |
          pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Build site
        run: |
          cd source                      
          mkdocs build -d ../docs       

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
          publish_branch: release

```

---

## Вывод

В результате работы получилось настроить CI/CD так, что сайт запускаеться и собирается автоматически



