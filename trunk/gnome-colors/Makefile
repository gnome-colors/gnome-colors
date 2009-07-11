PROJECT := gnome-colors
DOCS := AUTHORS ChangeLog COPYING README
SIZES := 16x16 22x22 24x24 32x32
THEME_FILES := $(wildcard themes/*)
LINKS_FILE := links
DIST_EXTRA := Makefile Palette.svg

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
DIST_PACKAGE := $(PROJECT)-$(VERSION)
SRC_PACKAGE := $(PROJECT)-src-$(VERSION)

ifeq ($(THEME_FILES),)
ALL_THEMES := $(shell find * -maxdepth 1 ! -wholename '*/.*' -type f -name index.theme -printf "%h ")
THEMES := $(filter-out src,$(filter-out gnome-colors-common,$(ALL_THEMES)))
TARBALLS := $(DIST_PACKAGE)
else
THEMES := $(notdir $(THEME_FILES))
TARBALLS := $(DIST_PACKAGE) $(SRC_PACKAGE)
endif
FIXED := $(shell find src/*x* ! -wholename '*/.*' -type f -name '*.svg' 2> /dev/null)
SCALABLE := $(shell find src/scalable ! -wholename '*/.*' -type f -name '*.svg' 2> /dev/null)
ICON_FILES := $(shell find src ! -wholename '*/.*' -type f -name '*.icon' 2> /dev/null)
COMMON_FILES := $(shell find gnome-colors-common ! -wholename '*/.*' -type f)
COMMON_LINKS := $(shell find gnome-colors-common ! -wholename '*/.*' -type l)
SRC_FILES := $(shell find src ! -wholename '*/.*' -type f 2> /dev/null)
ONESIZE_FILES := $(patsubst src/scalable/%,%,$(SCALABLE))
LINK_LIST := $(shell sed "s/ /-->/" $(LINKS_FILE) 2> /dev/null)
LINK_FILES := $(foreach link,$(LINK_LIST),$(lastword $(subst -->, ,$(link))))
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
SRC_DIST_FILES := \
	$(SRC_FILES) \
	$(DOCS) \
	$(DIST_EXTRA) \
	$(THEME_FILES) \
	$(LINKS_FILE)

config = $(shell grep "^$(2)=" themes/$(1) | sed "s/^$(2)=//")

ifeq ($(THEME_FILES),)
all:
else
all: $(DST_FILES) $(DST_LINKS)
endif

define theme_template
$(1)_c1 := $$(call config,$(1),LightFolderBase)
$(1)_c2 := $$(call config,$(1),LightBase)
$(1)_c3 := $$(call config,$(1),MediumBase)
$(1)_c4 := $$(call config,$(1),DarkStroke)
$(1)_sed := s/\#8fb3d9/$$($(1)_c1)/g;s/\#729fcf/$$($(1)_c2)/g;s/\#3465a4/$$($(1)_c3)/g;s/\#204a87/$$($(1)_c4)/g

$(1): $(call THEME_DST_FILES,$(1)) $(call THEME_DST_LINKS,$(1))

$(1)/index.theme: src/index.theme themes/$(1)
	mkdir -p $$(dir $$@)
	sed "s/@NAME@/$(call config,$(1),Name)/" $$< > $$@
endef

define png_template
$(1)/$(2).png: src/$(2).svg themes/$(1)
	mkdir -p $$(dir $$@)
	sed "$$($(1)_sed)" $$< | inkscape -f /dev/stdin -e $$@
endef

define svg_template
$(1)/$(2): src/$(2) themes/$(1)
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
	cp $$< $$@
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
	rm -f *.tar *.tar.bz2 *.tar.gz
ifneq ($(THEME_FILES),)
	rm -rf $(THEMES)
endif

BROKEN_LINKS = $(shell find * ! -wholename '*/.*' -type l ! -exec test -r {} \; -printf "%p ")

check:
	@for link in $(BROKEN_LINKS); do echo "Found broken link $$link"; done
	@test -z "$(BROKEN_LINKS)"

dist: gz

help:
	@echo "make targets:"
	@echo "    all               Does all or nothing (default target)."
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

bz2: $(addsuffix .tar.bz2,$(TARBALLS))

gz: $(addsuffix .tar.gz,$(TARBALLS))

# The tar command must be split, otherwise you get this error:
# make: execvp: /bin/sh: Argument list too long
$(DIST_PACKAGE).tar: $(DST_FILES) $(DOCS) $(DIST_EXTRA) $(DST_LINKS) $(COMMON_FILES) $(COMMON_LINKS)
	$(MAKE) fixperms
	tar -cf $@ $(DST_FILES) $(DOCS) $(DIST_EXTRA)
	tar -rf $@ $(DST_LINKS)
	tar -rf $@ $(COMMON_FILES)
	tar -rf $@ $(COMMON_LINKS)

$(DIST_PACKAGE).tar.bz2: $(DIST_PACKAGE).tar
	bzip2 -cz9 $< > $@

$(DIST_PACKAGE).tar.gz: $(DIST_PACKAGE).tar
	gzip -cn9 $< > $@

# The tar command must be split, otherwise you get this error:
# make: execvp: /bin/sh: Argument list too long
$(SRC_PACKAGE).tar: $(SRC_DIST_FILES) $(COMMON_FILES) $(COMMON_LINKS)
	$(MAKE) fixperms
	tar -cf $@ $(SRC_DIST_FILES)
	tar -rf $@ $(COMMON_FILES)
	tar -rf $@ $(COMMON_LINKS)

$(SRC_PACKAGE).tar.bz2: $(SRC_PACKAGE).tar
	bzip2 -cz9 $< > $@

$(SRC_PACKAGE).tar.gz: $(SRC_PACKAGE).tar
	gzip -cn9 $< > $@

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

$(foreach file,$(DST_FILES) $(COMMON_FILES),$(eval $(call install_template,$(file))))
$(foreach file,$(DST_LINKS) $(COMMON_LINKS),$(eval $(call install_links_template,$(file))))

define theme_install_template
install-$(1): uninstall-$(1) $(addprefix $(SYSTEM)/,$(call THEME_DST_FILES,$(1)) $(call THEME_DST_LINKS,$(1)))

user-install-$(1): user-uninstall-$(1) $(addprefix $(USER)/,$(call THEME_DST_FILES,$(1)) $(call THEME_DST_LINKS,$(1)))

uninstall-$(1):
	rm -rf $(SYSTEM)/$(1)

user-uninstall-$(1):
	rm -rf $(USER)/$(1)

UNINSTALL_TARGETS += uninstall-$(1) user-uninstall-$(1)
endef

$(foreach theme,$(THEMES) gnome-colors-common, $(eval $(call theme_install_template,$(theme))))

install: uninstall $(addprefix $(SYSTEM)/,$(DST_FILES) $(DST_LINKS) $(COMMON_FILES) $(COMMON_LINKS))

user-install: user-uninstall $(addprefix $(USER)/,$(DST_FILES) $(DST_LINKS) $(COMMON_FILES) $(COMMON_LINKS))

uninstall:
	rm -rf $(addprefix $(SYSTEM)/,$(THEMES) gnome-colors-common)

user-uninstall:
	rm -rf $(addprefix $(USER)/,$(THEMES) gnome-colors-common)

.PHONY: check clean fixperms help uninstall user-uninstall $(UNINSTALL_TARGETS)
