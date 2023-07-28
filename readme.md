Para rodar localmente:
```
sudo apt install hugo
git submodule init
git submodule update
hugo serve -D
```

convert from .tex to md using pandoc:

```
pandoc --bibliography article.bib -o output.md -t markdown_strict  --citeproc --standalone article.tex
```