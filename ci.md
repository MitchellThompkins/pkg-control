# CI Notes

## Why check-ci builds its own tarball

The canonical way to test an Octave package is `pkg install` followed by
`pkg test`. Both require an installable tarball, which `make dist` normally
produces.

`make dist` cannot run in the CI Docker image (`gnuoctave/octave`) because
the PDF documentation step fails. The `makeinfo --pdf` step invokes `pdfetex`,
which cannot include SVG images, and the manual references the project logo as
an SVG. No conversion toolchain (inkscape, rsvg-convert, etc.) is present in
the image.

The `check-ci` target works around this by building a doc-free tarball via the
`RELEASE_DIR_CI` make target. That target mirrors `RELEASE_DIR` (used by
`make dist`) but omits the three lines that copy the PDF, QCH, and logo into
the package directory.

## Git safe.directory in Docker

The CI container runs as root while the bind-mounted source tree is owned by
the host user. Git 2.35.2 and later refuse to operate on repositories owned
by a different user. The `RELEASE_DIR_CI` recipe passes the trust exception
via environment variables (`GIT_CONFIG_COUNT`, `GIT_CONFIG_KEY_0`,
`GIT_CONFIG_VALUE_0`) so no global git config file is modified.
