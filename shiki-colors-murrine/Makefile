PROJECT := shiki-colors-murrine
DOCS := AUTHORS ChangeLog COPYING README
THEMES := \
	Shiki-Brave \
	Shiki-Colors-Easy-Metacity \
	Shiki-Colors-Metacity \
	Shiki-Colors-Striped-Metacity \
	Shiki-Dust \
	Shiki-Human \
	Shiki-Illustrious \
	Shiki-Noble \
	Shiki-Wine \
	Shiki-Wise
WALLPAPER := Arc-Colors-Transparent-Wallpaper.png
DIST_EXTRA := Makefile userChrome.css

FILES := $(shell find $(THEMES) ! -wholename '*/.*' -type f)
USER := ~/.themes
SYSTEM := $(DESTDIR)/usr/share/themes
SYSTEM_WP := $(DESTDIR)/usr/share/backgrounds
DIST_FILES := $(FILES) $(DOCS) $(WALLPAPER) $(DIST_EXTRA)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

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

define install_template
$(SYSTEM)/$(1): $(1)
	install -D -m 644 $$< $$@

$(USER)/$(1): $(1)
	install -D -m 644 $$< $$@
endef

define wallpaper_template
$(SYSTEM_WP)/$(1): $(1)
	install -D -m 644 $$< $$@
endef

$(foreach file,$(FILES),$(eval $(call install_template,$(file))))
$(foreach file,$(WALLPAPER),$(eval $(call wallpaper_template,$(file))))

install: $(addprefix $(SYSTEM)/,$(FILES)) $(addprefix $(SYSTEM_WP)/,$(WALLPAPER))

user-install: $(addprefix $(USER)/,$(FILES))

uninstall:
	rm -rf $(addprefix $(SYSTEM)/,$(THEMES))
	rm -f $(addprefix $(SYSTEM_WP)/,$(WALLPAPER))

user-uninstall:
	rm -rf $(addprefix $(USER)/,$(THEMES))

.PHONY: clean help uninstall user-uninstall
