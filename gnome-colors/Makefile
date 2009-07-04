PROJECT := gnome-colors
DOCS := AUTHORS ChangeLog COPYING README
THEMES := \
	gnome-brave \
	gnome-colors-common \
	gnome-dust \
	gnome-human \
	gnome-noble \
	gnome-wine \
	gnome-wise
DIST_EXTRA := Makefile Palette.svg

FILES := $(shell find $(THEMES) -type d -a ! -wholename '*/.*' -exec sh -c 'if ls $$1/*.* > /dev/null 2>&1; then printf "$$1/*.* "; fi' {} {} \;)
USER := /home/$(shell whoami)/.icons
SYSTEM := $(DESTDIR)/usr/share/icons
DIST_FILES := $(FILES) $(DOCS) $(DIST_EXTRA)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

all:

clean:
	rm -f *.tar.bz2 *.tar.gz

dist: bz2

help:
	@echo "make targets:"
	@echo "    all               Does nothing (default target)."
	@echo "    clean             Deletes all files created by this makefile."
	@echo "    dist              Creates a distribution tar file for $(PROJECT)."
	@echo "    help              Displays this help."
	@echo "    bz2               Create a distribution .tar.bz2 file."
	@echo "    gz                Create a distribution .tar.gz file."
	@echo "    install           Installs $(PROJECT) system-wide."
	@echo "    uninstall         Removes $(PROJECT) from the system."
	@echo "    user-install      Installs $(PROJECT) into the user directory."
	@echo "    user-uninstall    Removes $(PROJECT) from the user directory."

bz2: $(PACKAGE).tar.bz2

gz: $(PACKAGE).tar.gz

$(PACKAGE).tar.bz2: $(DIST_FILES)
	tar -c $(DIST_FILES) | bzip2 -cz9 > $@

$(PACKAGE).tar.gz: $(DIST_FILES)
	tar -c $(DIST_FILES) | gzip -cn9 > $@

install: uninstall
	for DIR in $(patsubst %/*.*,%,$(FILES)); do mkdir -p $(SYSTEM)/$$DIR; done
	for FILE in $(FILES); do cp -d $$FILE $(SYSTEM)/$$FILE; done

uninstall:
	for THEME in $(THEMES); do rm -rf "$(SYSTEM)/$$THEME"; done

user-install: user-uninstall
	for DIR in $(patsubst %/*.*,%,$(FILES)); do mkdir -p $(USER)/$$DIR; done
	for FILE in $(FILES); do cp -d $$FILE $(USER)/$$FILE; done

user-uninstall:
	for THEME in $(THEMES); do rm -rf "$(USER)/$$THEME"; done

.PHONY: clean help install uninstall user-install user-uninstall
