PROJECT := shiki-colors
DOCS := AUTHORS ChangeLog COPYING README
# Spaces in filenames must be escaped with backslashes.
THEMES := \
	Shiki-Brave \
	Shiki-Colors\ Easy\ Metacity \
	Shiki-Colors\ Metacity \
	Shiki-Colors\ Striped\ Metacity \
	Shiki-Dust \
	Shiki-Human \
	Shiki-Noble \
	Shiki-Wine \
	Shiki-Wise
WALLPAPER := Arc-Colors\ Transparent\ Wallpaper.png
DIST_EXTRA := Makefile userChrome.css

FILES := $(shell find $(THEMES) \( -type f -o -type l \) -a ! -wholename '*/.*' -printf '%p\n' | sed 's/ /\\ /g;s/\n/ /g')
USER := /home/$(shell whoami)/.themes
SYSTEM := $(DESTDIR)/usr/share/themes
DIST_FILES := $(FILES) $(DOCS) $(WALLPAPER) $(DIST_EXTRA)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

# addprefix command for files containing escaped spaces.
ADDPREFIX = $(subst +,\ ,$(addprefix $(1),$(subst \ ,+,$(2))))

all:

clean:
	rm -f *.tar.bz2 *.tar.gz

dist: gz

help:
	@echo "make targets:"
	@echo "    all               Does nothing (default target)."
	@echo "    clean             Deletes all files created by this makefile."
	@echo "    fixperms          Fixes permissions of all files."
	@echo "    dist              Creates a distribution tar file for $(PROJECT)."
	@echo "    help              Displays this help."
	@echo "    bz2               Create a distribution .tar.bz2 file."
	@echo "    gz                Create a distribution .tar.gz file."
	@echo "    install           Installs $(PROJECT) system-wide."
	@echo "    uninstall         Removes $(PROJECT) from the system."
	@echo "    user-install      Installs $(PROJECT) into the user directory."
	@echo "    user-uninstall    Removes $(PROJECT) from the user directory."

fixperms:
	find * ! -wholename '*/.*' -a -type f -a ! -perm 644 -exec chmod 644 {} \; -printf "chmod 644 %p\n"

bz2: $(PACKAGE).tar.bz2

gz: $(PACKAGE).tar.gz

$(PACKAGE).tar.bz2: $(DIST_FILES)
	$(MAKE) fixperms
	tar -c $(DIST_FILES) | bzip2 -cz9 > $@

$(PACKAGE).tar.gz: $(DIST_FILES)
	$(MAKE) fixperms
	tar -c $(DIST_FILES) | gzip -cn9 > $@

install: uninstall
	for FILE in $(FILES); do install -D -m 644 "$$FILE" "$(SYSTEM)/$$FILE"; done
	for FILE in $(WALLPAPER); do install -D -m 644 "$$FILE" "$(DESTDIR)/usr/share/backgrounds/$$FILE"; done

user-install: user-uninstall
	for FILE in $(FILES); do install -D -m 644 "$$FILE" "$(USER)/$$FILE"; done

uninstall:
	rm -rf $(call ADDPREFIX,$(SYSTEM)/,$(THEMES))
	rm -f $(call ADDPREFIX,$(DESTDIR)/usr/share/backgrounds/,$(WALLPAPER))

user-uninstall:
	rm -rf $(call ADDPREFIX,$(USER)/,$(THEMES))

.PHONY: clean help install uninstall user-install user-uninstall
