PROJECT := gnome-colors
DOCS := AUTHORS ChangeLog COPYING README
THEMES := gnome-brave gnome-dust gnome-human gnome-noble gnome-wine gnome-wise
DIST_EXTRA := Makefile Palette.svg

FILES := $(shell find $(THEMES) -type d -a ! -wholename '*/.*' -exec sh -c 'if ls $$1/*.* > /dev/null 2>&1; then printf "$$1/*.* "; fi' {} {} \;)
COMMON_FILES := $(shell find gnome-colors-common -type d -a ! -wholename '*/.*' -exec sh -c 'if ls $$1/*.* > /dev/null 2>&1; then printf "$$1/*.* "; fi' {} {} \; 2> /dev/null)
USER := /home/$(shell whoami)/.icons
SYSTEM := $(DESTDIR)/usr/share/icons
DIST_FILES := $(FILES) $(DOCS) $(DIST_EXTRA)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

all:

clean:
	rm -f *.tar *.tar.bz2 *.tar.gz

dist: gz

help:
	@echo "make targets:"
	@echo "  all*            Does nothing."
	@echo "  clean           Delete all files created by this Makefile."
	@echo "  dist            Create a distribution tar file for this project."
	@echo "  help            Displays this help."
	@echo "  bz2             Create a distribution .tar.bz2 file."
	@echo "  gz              Create a distribution .tar.gz file."
	@echo "  install         Installs the complete package into the system (for all users)."
	@echo "  uninstall       Removes the package from the system."
	@echo "  user-install    Installs the complete package into the user directory."
	@echo "  user-uninstall  Removes the package from the user directory."
	@echo
	@echo "  * = default"

bz2: $(PACKAGE).tar.bz2

gz: $(PACKAGE).tar.gz

$(PACKAGE).tar: $(DIST_FILES) $(COMMON_FILES)
	tar -cf $@ --transform="s|^|$(PACKAGE)/|" $(DIST_FILES)
	for THEME in $(THEMES); do \
		tar -rf $@ --transform="s|^gnome-colors-common|$(PACKAGE)/$$THEME|" $(COMMON_FILES); \
	done

$(PACKAGE).tar.bz2: $(PACKAGE).tar
	bzip2 -cz9 $< > $@

$(PACKAGE).tar.gz: $(PACKAGE).tar
	gzip -cn9 $< > $@

install: uninstall
	for FILE in $(FILES); do \
		mkdir -p $$(dirname $(SYSTEM)/$$FILE); \
		cp -d $$FILE $(SYSTEM)/$$FILE; \
	done
ifneq ($(COMMON_FILES),)
	for THEME in $(THEMES); do \
		for FILE in $(COMMON_FILES); do \
			DEST=$$(echo $$FILE | sed "s|^gnome-colors-common|$$THEME|"); \
			mkdir -p $$(dirname $(SYSTEM)/$$DEST); \
			cp -d $$FILE $(SYSTEM)/$$DEST; \
		done; \
	done
endif

uninstall:
	for THEME in $(THEMES); do rm -rf "$(SYSTEM)/$$THEME"; done

user-install: user-uninstall
	for FILE in $(FILES); do \
		mkdir -p $$(dirname $(USER)/$$FILE); \
		cp -d $$FILE $(USER)/$$FILE; \
	done
ifneq ($(COMMON_FILES),)
	for THEME in $(THEMES); do \
		for FILE in $(COMMON_FILES); do \
			DEST=$$(echo $$FILE | sed "s|^gnome-colors-common|$$THEME|"); \
			mkdir -p $$(dirname $(USER)/$$DEST); \
			cp -d $$FILE $(USER)/$$DEST; \
		done; \
	done
endif

user-uninstall:
	for THEME in $(THEMES); do rm -rf "$(USER)/$$THEME"; done

.PHONY: clean help install uninstall user-install user-uninstall
