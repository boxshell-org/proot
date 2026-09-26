# PRoot documentation build.
#
# Single source of truth: src/cli/proot.h defines the command line
# (options, descriptions, recommended bindings); doc/proot.md holds the
# maintained prose.  generate.py merges them into MANUAL.md (Markdown)
# and proot.1 (troff).  `make check` fails if the checked-in copies are
# stale, so docs cannot drift from `proot --help`.

PYTHON    ?= python3
GENERATED := MANUAL.md proot.1
INPUTS    := proot.md generate.py ../src/cli/proot.h

all: $(GENERATED)

$(GENERATED): $(INPUTS)
	$(PYTHON) generate.py

check: $(INPUTS)
	@$(PYTHON) generate.py --check

install: all
	install -Dm644 proot.1 $(DESTDIR)$(PREFIX)/share/man/man1/proot.1

clean:
	rm -f $(GENERATED)

.PHONY: all check install clean
