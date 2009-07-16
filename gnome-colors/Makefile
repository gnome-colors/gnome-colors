PROJECT := gnome-colors-src
DOCS := AUTHORS ChangeLog COPYING README
SIZES := 16x16 22x22 24x24 32x32
THEME_FILES := $(filter-out %~,$(wildcard themes/*))
LINKS_FILE := links
DIST_EXTRA := Makefile Palette.svg

gnome-colors_DIST_COMMON := yes
$(PROJECT)_DIST_COMMON := yes

############################### Global variables ###############################

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)

ifeq ($(THEME_FILES),)
ALL_THEMES := $(filter-out src,$(shell find * -maxdepth 1 ! -wholename '*/.*' -type f -name index.theme -printf "%h "))
THEMES := $(filter-out gnome-colors-common,$(ALL_THEMES))
THEME_BUILD_FILES = $(shell find $(1) ! -wholename '*/.*' -type f)
THEME_BUILD_LINKS = $(shell find $(1) ! -wholename '*/.*' -type l)
else
THEMES := $(notdir $(THEME_FILES))
ALL_THEMES := $(THEMES) gnome-colors-common
config = $(shell grep "^$(2)=" themes/$(1) | sed "s/^$(2)=//")
FIXED := $(shell find src/*x* ! -wholename '*/.*' -type f -name '*.svg')
SCALABLE := $(shell find src/scalable ! -wholename '*/.*' -type f -name '*.svg')
ICON_FILES := $(shell find src ! -wholename '*/.*' -type f -name '*.icon')
ONESIZE_FILES := $(patsubst src/scalable/%,%,$(SCALABLE))
LINK_LIST := $(shell sed "s/ /-->/" $(LINKS_FILE))
LINK_FILES := $(foreach link,$(LINK_LIST),$(lastword $(subst -->, ,$(link))))
THEME_BUILD_FILES = $(1)/index.theme \
	$(patsubst src/%,$(1)/%,$(SCALABLE)) \
	$(addprefix $(1)/,$(FIXED:src/%.svg=%.png)) \
	$(addprefix $(1)/24x24/,$(ONESIZE_FILES:%.svg=%.png)) \
	$(patsubst src/%,$(1)/%,$(ICON_FILES))
THEME_BUILD_LINKS = $(addprefix $(1)/scalable/,$(LINK_FILES:%.png=%.svg)) \
	$(foreach size,$(SIZES),$(addprefix $(1)/$(size)/,$(LINK_FILES)))
endif

THEME_BUILD = $(THEME_BUILD_FILES) $(THEME_BUILD_LINKS)
BUILD_FILES := $(foreach theme,$(THEMES),$(call THEME_BUILD_FILES,$(theme)))
BUILD_LINKS := $(foreach theme,$(THEMES),$(call THEME_BUILD_LINKS,$(theme)))
BUILD := $(BUILD_FILES) $(BUILD_LINKS)
COMMON_FILES := $(shell find gnome-colors-common ! -wholename '*/.*' -type f)
COMMON_LINKS := $(shell find gnome-colors-common ! -wholename '*/.*' -type l)
COMMON := $(COMMON_FILES) $(COMMON_LINKS)

################################### Building ###################################

# all should be the first target
all:

ifneq ($(THEME_FILES),)
all: $(BUILD)
endif

define theme_template
$(1)_c1 := $$(call config,$(1),LightFolderBase)
$(1)_c2 := $$(call config,$(1),LightBase)
$(1)_c3 := $$(call config,$(1),MediumBase)
$(1)_c4 := $$(call config,$(1),DarkStroke)
$(1)_sed := s/\#8fb3d9/$$($(1)_c1)/g;s/\#729fcf/$$($(1)_c2)/g;s/\#3465a4/$$($(1)_c3)/g;s/\#204a87/$$($(1)_c4)/g

$(1): $(call THEME_BUILD,$(1))

$(1)/index.theme: src/index.theme themes/$(1)
	@if test ! -d $(1); then mkdir -p $(1); echo "mkdir -p $(1)"; fi
	sed "s/@NAME@/$(call config,$(1),Name)/" $$< > $$@
endef

ifneq ($(THEME_FILES),)
$(foreach theme,$(THEMES),$(eval $(call theme_template,$(theme))))
endif

define png_template
$(1)/$(2).png: src/$(2).svg themes/$(1)
	@if test ! -d $$(dir $$@); then mkdir -p $$(dir $$@); echo "mkdir -p $$(dir $$@)"; fi
	sed "$$($(1)_sed)" $$< | inkscape --without-gui -f /dev/stdin -e "$$(CURDIR)/$$@"
endef

define svg_template
$(1)/$(2): src/$(2) themes/$(1)
	@if test ! -d $$(dir $$@); then mkdir -p $$(dir $$@); echo "mkdir -p $$(dir $$@)"; fi
	sed "$$($(1)_sed)" $$< > $$@
endef

define 24_template
$(1)/24x24/$(2): $(1)/22x22/$(2)
	@if test ! -d $$(dir $$@); then mkdir -p $$(dir $$@); echo "mkdir -p $$(dir $$@)"; fi
	convert -bordercolor Transparent -border 1x1 $$< $$@
endef

define link_template
$(1)/$(3):
	@if test ! -d $$(dir $$@); then mkdir -p $$(dir $$@); echo "mkdir -p $$(dir $$@)"; fi
	ln -sf $(2) $$@
endef

define icon_template
$(2): $(1)
	@if test ! -d $$(dir $$@); then mkdir -p $$(dir $$@); echo "mkdir -p $$(dir $$@)"; fi
	cp $$< $$@
endef

first = $(firstword $(subst -->, ,$(1)))
last = $(lastword $(subst -->, ,$(1)))

$(foreach theme,$(THEMES), \
	$(foreach file,$(patsubst src/%.svg,%,$(FIXED)),$(eval $(call png_template,$(theme),$(file)))) \
	$(foreach file,$(patsubst src/%,%,$(SCALABLE)),$(eval $(call svg_template,$(theme),$(file)))) \
	$(foreach file,$(ONESIZE_FILES:%.svg=%.png),$(eval $(call 24_template,$(theme),$(file)))) \
	$(foreach link,$(LINK_LIST),$(foreach size,$(SIZES),$(eval $(call link_template \
		,$(theme)/$(size),$(call first,$(link)),$(call last,$(link)))))) \
	$(foreach link,$(LINK_LIST),$(eval $(call link_template \
		,$(theme)/scalable,$(patsubst %.png,%.svg,$(call first,$(link))),$(patsubst %.png,%.svg,$(call last,$(link)))))) \
	$(foreach icon,$(ICON_FILES),$(eval $(call icon_template,$(icon),$(patsubst src/%,$(theme)/%,$(icon))))) \
)

help::
	@echo "make targets:"
	@echo "    all                     Does all or nothing (default target)."

#################################### Cleanup ###################################

clean:
	rm -f *.tar *.tar.bz2 *.tar.gz
ifneq ($(THEME_FILES),)
	rm -rf $(THEMES)
endif

help::
	@echo "    clean                   Deletes all files created by this makefile."

.PHONY: clean

################################# Miscellaneous ################################

BROKEN_LINKS = $(shell find * ! -wholename '*/.*' -type l ! -exec test -r {} \; -printf "%p ")

check:
	@for link in $(BROKEN_LINKS); do echo "Found broken link $$link"; done
	@test -z "$(BROKEN_LINKS)"

fixperms:
	find * ! -wholename '*/.*' -a -type f -a ! -perm 644 -exec chmod 644 {} \; -printf "chmod 644 %p\n"

themes:
	@echo "Available themes:"
	@for theme in $(sort $(ALL_THEMES)); do echo $$theme; done

help::
	@echo "    check                   Performs self tests."
	@echo "    fixperms                Fixes permissions of icon files."
	@echo "    help                    Displays this help."
	@echo "    themes                  Lists all available themes."

.PHONY: check fixperms help themes

################################# Distribution #################################

ifneq ($(THEME_FILES),)

DISTRIBUTIONS := $(PROJECT) \
	$(sort $(foreach theme,$(THEMES),$(call config,$(theme),Distribution)))
$(PROJECT)_DIST := \
	$(shell find src ! -wholename '*/.*' -type f) \
	$(THEME_FILES) \
	$(LINKS_FILE)

define dist_theme_template
$(call config,$(1),Distribution)-$(VERSION).tar: $(call THEME_BUILD,$(1))
endef

define dist_template
# The tar command must be split, otherwise you get this error:
# make: execvp: /bin/sh: Argument list too long
$(1)-$(VERSION).tar: $$(DOCS) $$(DIST_EXTRA) $$($(1)_DIST)
	$$(MAKE) fixperms
	tar -cf $$@ $$(filter-out $$(COMMON),$$^)
ifeq ($($(1)_DIST_COMMON),yes)
	tar -rf $$@ $$(COMMON_FILES)
	tar -rf $$@ $$(COMMON_LINKS)

$(1)-$(VERSION).tar: $(COMMON)
endif

$(1)-$(VERSION).tar.bz2: $(1)-$(VERSION).tar
	bzip2 -cz9 $$< > $$@

$(1)-$(VERSION).tar.gz: $(1)-$(VERSION).tar
	gzip -cn9 $$< > $$@
endef

$(foreach theme,$(THEMES),$(eval $(call dist_theme_template,$(theme))))
$(foreach dist,$(DISTRIBUTIONS),$(eval $(call dist_template,$(dist))))

dist: gz

dist-src: $(PROJECT)-$(VERSION).tar.gz

bz2: $(addsuffix -$(VERSION).tar.bz2,$(DISTRIBUTIONS))

gz: $(addsuffix -$(VERSION).tar.gz,$(DISTRIBUTIONS))

help::
	@echo "    dist                    Creates distribution tar files."
	@echo "    dist-src                Creates a source distribution tar file."
	@echo "    bz2                     Create distribution .tar.bz2 files."
	@echo "    gz                      Create distribution .tar.gz files."

endif

################################# Installation #################################

USER := /home/$(shell whoami)/.icons
SYSTEM := $(DESTDIR)/usr/share/icons

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

$(foreach file,$(BUILD_FILES) $(COMMON_FILES),$(eval $(call install_template,$(file))))
$(foreach file,$(BUILD_LINKS) $(COMMON_LINKS),$(eval $(call install_links_template,$(file))))

define theme_install_template
install-$(1): $$(addprefix $$(SYSTEM)/,$(call THEME_BUILD,$(1)))

user-install-$(1): $$(addprefix $$(USER)/,$(call THEME_BUILD,$(1)))
endef

define theme_uninstall_template
uninstall-$(1):
	rm -rf $$(SYSTEM)/$(1)

user-uninstall-$(1):
	rm -rf $$(USER)/$(1)

.PHONY: uninstall-$(1) user-uninstall-$(1)
endef

$(foreach theme,$(THEMES),$(eval $(call theme_install_template,$(theme))))
$(foreach theme,$(ALL_THEMES),$(eval $(call theme_uninstall_template,$(theme))))

install-gnome-colors-common: $(addprefix $(SYSTEM)/,$(COMMON))

user-install-gnome-colors-common: $(addprefix $(USER)/,$(COMMON))

install: $(addprefix install-,$(ALL_THEMES))

user-install: $(addprefix user-install-,$(ALL_THEMES))

uninstall:
	rm -rf $(addprefix $(SYSTEM)/,$(ALL_THEMES))

user-uninstall:
	rm -rf $(addprefix $(USER)/,$(ALL_THEMES))

help::
	@echo "    install                 Installs all themes system-wide."
	@echo "    uninstall               Removes all themes from the system."
	@echo "    user-install            Installs all themes into the user directory."
	@echo "    user-uninstall          Removes all themes from the user directory."
	@echo "    install-THEME           Installs THEME system-wide."
	@echo "    uninstall-THEME         Removes THEME from the system."
	@echo "    user-install-THEME      Installs THEME into the user directory."
	@echo "    user-uninstall-THEME    Removes THEME from the user directory."

.PHONY:  uninstall user-uninstall
