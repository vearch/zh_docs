# zh_docs
Chinese version docs for vearch

## local test

pip install sphinx sphinx-rtd-theme sphinx-copybutton

source venv/bin/activate && sphinx-build -b html docs/source docs/build/html

cd docs/build/html && python3 -m http.server 8080