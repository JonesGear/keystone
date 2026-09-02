# Storage behind one module
Dated 2026-09-01. Replaces: nothing.

## Decisions
- **Only app/storage.py touches the filesystem. Callers use save/get/get_path/delete with relative keys.**
  Reason: swapping to object storage later must be one file.
  Silent default: key = `<kind>/<owner ids>/<uuid or id>/<safe filename>`.
  Check: `grep -rnE "(^|[^.[:alnum:]_])open\(|\bPath\(" app/ --include=*.py | grep -v storage.py | wc -l` is 0 (library methods like `fitz.open(` do not match).
- **Key builders live in storage.py (`key_for_<kind>(...)`); callers never format keys.**
  Reason: keys built by f-string at two call sites with an unsanitized filename.
  Silent default: the builder sanitizes the filename.
  Check: `grep -rn "storage.save(" app/ | grep -v "key_for_" | wc -l` is 0.
- **Resolved paths must stay under the root; extensions and sizes are allow-listed at the boundary.**
  Reason: traversal and 26MB books.
  Silent default: reject unknown extensions with a flash, cap per kind.
  Check: `pytest tests/test_storage.py::test_traversal_rejected`.
- **Uploads are never deleted by student action; deletion is admin cleanup only.**
  Reason: uploads are the source record.
  Silent default: replace = new row + keep old file.
  Check: review.

## Layout
app/storage.py · volume mounted at UPLOAD_ROOT

## Not covered
Volume names (docker-compose-stack), what is stored (the spec).
