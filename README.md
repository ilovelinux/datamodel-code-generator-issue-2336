## [datamodel-code-generator](https://github.com/koxudaxi/datamodel-code-generator/) reproducible example for [issue #2336](https://github.com/koxudaxi/datamodel-code-generator/issues/2336)

|Inputs|Outputs|
|-|-|
|[inputs/array-commons.schema.json](inputs/array-commons.schema.json)|-|
|[inputs/products.schema.json](inputs/products.schema.json)|-|

### Description of the problem

- We define an object in [inputs/array-commons.schema.json](inputs/commons.schema.json)
- We `ref` the object `array-commons.schema.json#/$defs/$defaultArray` in [inputs/products.schema.json](inputs/products.schema.json)

### Goal

We want to generate a **a model of any supported type** (pydantic v1, pydantic v2, etc..) for `products`.

### Issue

`datamodel-code-generator` doesn't support references to schemas with both minus (`-`) and dots (`.`) because file names are not sanitized. This lead to a crash because invalid python code is passed to black, which raises an exception.

As you can see from the last line of the traceback, it tries to import `array-commons`, which is an invalid module name in Python.

```pytb
$ datamodel-codegen
Traceback (most recent call last):
  File "<path>/.venv/lib/python3.13/site-packages/datamodel_code_generator/__main__.py", line 452, in main
    generate(
    ~~~~~~~~^
        input_=config.url or config.input or sys.stdin.read(),
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    ...<67 lines>...
        no_alias=config.no_alias,
        ^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "<path>/.venv/lib/python3.13/site-packages/datamodel_code_generator/__init__.py", line 481, in generate
    results = parser.parse()
  File "<path>/.venv/lib/python3.13/site-packages/datamodel_code_generator/parser/base.py", line 1309, in parse
    body = code_formatter.format_code(body)
  File "<path>/.venv/lib/python3.13/site-packages/datamodel_code_generator/format.py", line 188, in format_code
    code = self.apply_black(code)
  File "<path>/.venv/lib/python3.13/site-packages/datamodel_code_generator/format.py", line 196, in apply_black
    return black.format_str(
           ~~~~~~~~~~~~~~~~^
        code,
        ^^^^^
        mode=self.black_mode,
        ^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "src/black/__init__.py", line 1208, in format_str
  File "src/black/__init__.py", line 1222, in _format_str_once
  File "src/black/parsing.py", line 98, in lib2to3_parse
black.parsing.InvalidInput: Cannot parse for target version Python 3.9: 5:12: from ..array-commons import schema
```

This issue was (partially) fixed in:
- https://github.com/koxudaxi/datamodel-code-generator/issues/131 <- Match this issue
- https://github.com/koxudaxi/datamodel-code-generator/issues/1141 <- Partially match this issue (no dots)

May be related to:
- #2332 

**It works for references without any dot (`.`)**

### Generate models

```sh
$ pip install -r requirements.txt
$ datamodel-codegen
```