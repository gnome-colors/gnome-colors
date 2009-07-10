PROJECT := colorizeme
DOCS := 
SRC_SIZES := \
	src/16x16 \
	src/22x22 \
	src/32x32
THEME_FILES := $(wildcard themes/*)
LINKS_FILE := links
DIST_EXTRA := Makefile

THEMES := $(notdir $(THEME_FILES))
FIXED := $(shell find $(SRC_SIZES) ! -wholename '*/.*' -type f -name '*.svg')
SCALABLE := $(shell find src/scalable ! -wholename '*/.*' -type f -name '*.svg')
ICON_FILES := $(shell find src ! -wholename '*/.*' -type f -name '*.icon')
SRC_FILES := $(shell find src ! -wholename '*/.*' -type f)
ONESIZE_FILES := $(patsubst src/scalable/%,%,$(SCALABLE))
LINK_LIST := $(shell sed "s/ /-->/" $(LINKS_FILE))
LINK_FILES := $(foreach link,$(LINK_LIST),$(lastword $(subst -->, ,$(link))))
SIZES := $(notdir $(SRC_SIZES)) 24x24
THEME_DST_FILES = $(1)/index.theme \
	$(patsubst src/%,$(1)/%,$(SCALABLE)) \
	$(addprefix $(1)/,$(FIXED:src/%.svg=%.png)) \
	$(addprefix $(1)/24x24/,$(ONESIZE_FILES:%.svg=%.png)) \
	$(patsubst src/%,$(1)/%,$(ICON_FILES))
THEME_DST_LINKS = $(addprefix $(1)/scalable/,$(LINK_FILES:%.png=%.svg)) \
	$(foreach size,$(SIZES),$(addprefix $(1)/$(size)/,$(LINK_FILES)))
DST_FILES := $(foreach theme,$(THEMES),$(call THEME_DST_FILES,$(theme)))
DST_LINKS := $(foreach theme,$(THEMES),$(call THEME_DST_LINKS,$(theme)))
USER := /home/$(shell whoami)/.icons
SYSTEM := $(DESTDIR)/usr/share/icons
DIST_FILES := $(SRC_FILES) $(DOCS) $(DIST_EXTRA) $(THEME_FILES) $(LINKS_FILE)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

config = $(shell grep "^$(2)=" themes/$(1) | sed "s/^$(2)=//")

all: $(DST_FILES) $(DST_LINKS)

define theme_template
$(1)_c1 := $$(call config,$(1),LightFolderBase)
$(1)_c2 := $$(call config,$(1),LightBase)
$(1)_c3 := $$(call config,$(1),MediumBase)
$(1)_c4 := $$(call config,$(1),DarkStroke)
$(1)_sed := s/\#8fb3d9/$$($(1)_c1)/g;s/\#729fcf/$$($(1)_c2)/g;s/\#3465a4/$$($(1)_c3)/g;s/\#204a87/$$($(1)_c4)/g

$(1): $(call THEME_DST_FILES,$(1)) $(call THEME_DST_LINKS,$(1))

$(1)/index.theme: src/index.theme
	mkdir -p $$(dir $$@)
	sed "s/@NAME@/$(call config,$(1),Name)/" $$< > $$@
endef

define png_template
$(1)/$(2).png: src/$(2).svg
	mkdir -p $$(dir $$@)
	sed "$$($(1)_sed)" $$< | inkscape -f /dev/stdin -e $$@
endef

define svg_template
$(1)/$(2): src/$(2)
	mkdir -p $$(dir $$@)
	sed "$$($(1)_sed)" $$< > $$@
endef

define 24_template
$(1)/24x24/$(2): $(1)/22x22/$(2)
	mkdir -p $$(dir $$@)
	convert -bordercolor Transparent -border 1x1 $$< $$@
endef

define link_template
$(1)/$(3):
	mkdir -p $$(dir $$@)
	ln -sf $(2) $$@
endef

define icon_template
$(2): $(1)
	mkdir -p $$(dir $$@)
	cp $$^ $$@
endef

first = $(firstword $(subst -->, ,$(1)))
last = $(lastword $(subst -->, ,$(1)))

$(foreach theme,$(THEMES), \
	$(eval $(call theme_template,$(theme))) \
	$(foreach file,$(patsubst src/%.svg,%,$(FIXED)),$(eval $(call png_template,$(theme),$(file)))) \
	$(foreach file,$(patsubst src/%,%,$(SCALABLE)),$(eval $(call svg_template,$(theme),$(file)))) \
	$(foreach file,$(ONESIZE_FILES:%.svg=%.png),$(eval $(call 24_template,$(theme),$(file)))) \
	$(foreach link,$(LINK_LIST),$(foreach size,$(SIZES),$(eval $(call link_template \
		,$(theme)/$(size),$(call first,$(link)),$(call last,$(link)))))) \
	$(foreach link,$(LINK_LIST),$(eval $(call link_template \
		,$(theme)/scalable,$(patsubst %.png,%.svg,$(call first,$(link))),$(patsubst %.png,%.svg,$(call last,$(link)))))) \
	$(foreach icon,$(ICON_FILES),$(eval $(call icon_template,$(icon),$(patsubst src/%,$(theme)/%,$(icon))))) \
)

clean:
	rm -f *.tar.bz2 *.tar.gz
	rm -rf $(THEMES)

BROKEN_LINKS = $(shell find * ! -wholename '*/.*' -type l ! -exec test -r {} \; -printf "%p ")

check:
	@for link in $(BROKEN_LINKS); do echo "Found broken link $$link"; done
	@test -z "$(BROKEN_LINKS)"

dist: gz

help:
	@echo "make targets:"
	@echo "    all               Does 42 (default target)."
	@echo "    check             Performs self tests."
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

define install_links_template
$(SYSTEM)/$(1): $(1)
	cp -d $$< $$@

$(USER)/$(1): $(1)
	cp -d $$< $$@
endef

$(foreach file,$(DST_FILES),$(eval $(call install_template,$(file))))
$(foreach file,$(DST_LINKS),$(eval $(call install_links_template,$(file))))

install: uninstall $(addprefix $(SYSTEM)/,$(DST_FILES) $(DST_LINKS))

user-install: user-uninstall $(addprefix $(USER)/,$(DST_FILES) $(DST_LINKS))

uninstall:
	rm -rf $(addprefix $(SYSTEM)/,$(THEMES))

user-uninstall:
	rm -rf $(addprefix $(USER)/,$(THEMES))

.PHONY: check clean fixperms help install uninstall user-install user-uninstall
