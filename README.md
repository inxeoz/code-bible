# code-bible lint

Lint Python files against the rules in `rules.md`.

## Usage

```bash
python lint.py [path ...]            # files or directories (default: .)
python lint.py --format json .
python lint.py --max-nesting 2 file.py
cat file.py | python lint.py -
```

Exit status is `1` when violations are found, `0` otherwise.

Equivalent entry points: `linter.py`, `check.py`, `analyze.py`, `code_lint.py`, etc.

## Programmatic API

```python
from lint import lint_file, lint_source

violations = lint_file("app.py")
violations = lint_source(source_code, "app.py")
```

Each violation has `file`, `line`, `col`, `code`, `rule`, `message`.
