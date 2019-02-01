## Code Documentation
To browse the code more easily, do

```
mkdir build
cd build
cmake .. -DBUILD_DOCS=On
make doc
```

and then point your browser to `doc/html/index.html`.

Note that documentation generates class diagrams using [Graphviz](https://www.graphviz.org). For best results, ensure that it is installed beforehand.
