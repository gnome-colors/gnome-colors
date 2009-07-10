PROJECT := colorizeme
DOCS := 
SRC_SIZES := \
	16x16 \
	22x22 \
	32x32
DIST_EXTRA := Makefile $(wildcard *.colorizeme)

THEMES := $(basename $(wildcard *.colorizeme))
FIXED := $(shell find $(SRC_SIZES) ! -wholename '*/.*' -type f -name '*.svg')
SCALABLE := $(shell find scalable ! -wholename '*/.*' -type f -name '*.svg')
SRC_FILES := $(FIXED) $(SCALABLE)
ICON_FILES := $(patsubst scalable/%,%,$(SCALABLE))
DST_FILES := $(foreach theme,$(THEMES),\
	$(addprefix $(theme)/,$(SCALABLE)) \
	$(addprefix $(theme)/,$(FIXED:%.svg=%.png)) \
	$(addprefix $(theme)/24x24/,$(ICON_FILES:%.svg=%.png)))
USER := /home/$(shell whoami)/.icons
SYSTEM := $(DESTDIR)/usr/share/icons
DIST_FILES := $(SRC_FILES) $(DOCS) $(DIST_EXTRA)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

all: $(DST_FILES)

%.png: %.svg
	inkscape -f $< -e $@

define svg_template
$(1)/$(2): $(2)
	mkdir -p $$(dir $$@)
	sed "$(shell cat $(1).colorizeme)" $$< > $$@
endef

define 24_template
$(1)/24x24/$(2): $(1)/22x22/$(2)
	mkdir -p $$(dir $$@)
	convert -bordercolor Transparent -border 1x1 $$< $$@
endef

$(foreach theme,$(THEMES),$(foreach file,$(SRC_FILES),$(eval $(call svg_template,$(theme),$(file)))))
$(foreach theme,$(THEMES),$(foreach file,$(ICON_FILES:%.svg=%.png),$(eval $(call 24_template,$(theme),$(file)))))

clean:
	rm -f *.tar.bz2 *.tar.gz
	rm -rf $(THEMES)

dist: gz

help:
	@echo "make targets:"
	@echo "    all               Does 42 (default target)."
	@echo "    clean             Deletes all files created by this makefile."
	@echo "    dist              Creates a distribution tar file for $(PROJECT)."
	@echo "    fixperms          Fixes permissions of icon files."
	@echo "    help              Displays this help."
	@echo "    bz2               Create a distribution .tar.bz2 file."
	@echo "    gz                Create a distribution .tar.gz file."
	@echo "    install           Installs all themes system-wide."
	@echo "    uninstall         Removes all themes from the system."
	@echo "    user-install      Installs all themes into the user directory."
	@echo "    user-uninstall    Removes all themes from the user directory."

fixperms:
	find * ! -wholename '*/.*' -a -type f -a ! -perm 644 -exec chmod 644 {} \; -printf "chmod 644 %p\n"

bz2: $(PACKAGE).tar.bz2

gz: $(PACKAGE).tar.gz

$(PACKAGE).tar.bz2: fixperms $(DIST_FILES)
	tar -c $(DIST_FILES) | bzip2 -cz9 > $@

$(PACKAGE).tar.gz: fixperms $(DIST_FILES)
	tar -c $(DIST_FILES) | gzip -cn9 > $@

define install_template
$(SYSTEM)/$(1): $(1)
	install -D -m 644 $$< $$@

$(USER)/$(1): $(1)
	install -D -m 644 $$< $$@
endef

$(foreach file,$(DST_FILES),$(eval $(call install_template,$(file))))

install: uninstall $(addprefix $(SYSTEM)/,$(DST_FILES))

user-install: user-uninstall $(addprefix $(USER)/,$(DST_FILES))

uninstall:
	rm -rf $(addprefix $(SYSTEM)/,$(THEMES))

user-uninstall:
	rm -rf $(addprefix $(USER)/,$(THEMES))

.PHONY: clean fixperms help install uninstall user-install user-uninstall
