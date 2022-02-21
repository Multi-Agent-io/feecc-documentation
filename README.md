```
git clone https://github.com/Multi-Agent-io/feecc-documentation
cd feecc-documentation
docker pull squidfunk/mkdocs-material
docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```