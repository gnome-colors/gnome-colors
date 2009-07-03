PROJECT := arc-colors
DOCS := AUTHORS ChangeLog COPYING README
THEMES := \
	GDM-Themes/Arc-Brave \
	GDM-Themes/Arc-Dust \
	GDM-Themes/Arc-Human \
	GDM-Themes/Arc-Noble \
	GDM-Themes/Arc-Wine \
	GDM-Themes/Arc-Wise \
	GDM-UserList-Themes/Arc-Brave-Userlist \
	GDM-UserList-Themes/Arc-Dust-Userlist \
	GDM-UserList-Themes/Arc-Human-Userlist \
	GDM-UserList-Themes/Arc-Noble-Userlist \
	GDM-UserList-Themes/Arc-Wine-Userlist \
	GDM-UserList-Themes/Arc-Wise-Userlist
WALLPAPER := \
	Arc-Brave.png \
	Arc-Dust.jpg \
	Arc-Human.png \
	Arc-Noble.png \
	Arc-Wine.png \
	Arc-Wise.png
DIST_EXTRA := Makefile Sources

THEME_TARBALLS := $(addsuffix .tar.bz2,$(THEMES))
FILES := $(shell find $(THEMES) \( -type f -o -type l \) -a ! -wholename '*/.*' 2> /dev/null)
DIST_EXTRA_FILES := $(shell find $(DIST_EXTRA) \( -type f -o -type l \) -a ! -wholename '*/.*')
DIST_FILES := $(THEME_TARBALLS) $(DOCS) $(WALLPAPER) $(DIST_EXTRA_FILES)

VERSION := $(shell head -n 1 ChangeLog | cut -d ' ' -f 1)
PACKAGE := $(PROJECT)-$(VERSION)

all:

clean:
	rm -f *.tar.bz2 *.tar.gz

distclean: clean
	for THEME in $(THEMES); do \
		if test -f $${THEME}.tar.bz2; then \
			rm -rf $${THEME}; \
		fi; \
	done

svnclean: clean
	for THEME in $(THEMES); do \
		if test -d $${THEME}; then \
			rm -f $${THEME}.tar.bz2; \
		fi; \
	done

dist: gz

help:
	@echo "make targets:"
	@echo "    all          Does nothing (default target)."
	@echo "    clean        Deletes all files created by this makefile."
	@echo "    distclean    Removes all files, which are not part of the distribution."
	@echo "    svnclean     Removes all files, which are not part of the repository."
	@echo "    dist         Creates a distribution tar file for $(PROJECT)."
	@echo "    help         Displays this help."
	@echo "    bz2          Create a distribution .tar.bz2 file."
	@echo "    gz           Create a distribution .tar.gz file."
	@echo "    install      Installs $(PROJECT) system-wide."
	@echo "    uninstall    Removes $(PROJECT) from the system."

bz2: $(PACKAGE).tar.bz2

gz: $(PACKAGE).tar.gz

$(THEME_TARBALLS): %.tar.bz2: $(FILES)
	tar -c --transform='s|^[^/]*/||' $(filter $(patsubst %.tar.bz2,%,$@)%,$^) | bzip2 -cz9 > $@

$(PACKAGE).tar.bz2: $(DIST_FILES)
	tar -c --transform='s|^|$(PACKAGE)/|' $^ | bzip2 -cz9 > $@

$(PACKAGE).tar.gz: $(DIST_FILES)
	tar -c --transform='s|^|$(PACKAGE)/|' $^ | gzip -cn9 > $@

install: uninstall
# extract tarballs
	for THEME in $(THEMES); do \
		if test ! -d $$THEME; then \
			tar -xf $${THEME}.tar.bz2 -C $$(dirname $${THEME}.tar.bz2); \
		fi; \
	done
# install GDM themes
	for FILE in $$(find $(THEMES) \( -type f -o -type l \) -a ! -wholename '*/.*'); do \
		THEME=$$(echo $$FILE | sed 's|^[^/]*/||'); \
		install -D -m 644 "$$FILE" "$(DESTDIR)/usr/share/gdm/themes/$$THEME"; \
	done
# install wallpapers
	for FILE in $(WALLPAPER); do \
		install -D -m 644 "$$FILE" "$(DESTDIR)/usr/share/backgrounds/$$FILE"; \
		THEME=$$(echo $$FILE | sed 's/\.[^\.]*$$//'); \
		EXTENSION=$$(echo $$FILE | sed 's/.*\.//g'); \
		ln -sf ../../../backgrounds/$$FILE $(DESTDIR)/usr/share/gdm/themes/$$THEME/bg.$$EXTENSION; \
		ln -sf ../../../backgrounds/$$FILE $(DESTDIR)/usr/share/gdm/themes/$${THEME}-Userlist/bg.$$EXTENSION; \
	done

uninstall:
	for DIR in $(THEMES); do \
		THEME=$$(echo $$DIR | sed 's|^[^/]*/||'); \
		rm -rf "$(DESTDIR)/usr/share/gdm/themes/$$THEME"; \
	done
	for FILE in $(WALLPAPER); do rm -f "$(DESTDIR)/usr/share/backgrounds/$$FILE"; done

.PHONY: clean distclean help install svnclean uninstall
