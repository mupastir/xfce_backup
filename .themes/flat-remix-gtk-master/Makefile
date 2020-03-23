# GNU make is required to run this file. To install on *BSD, run:
#   gmake PREFIX=/usr/local install

PREFIX ?= /usr
IGNORE ?=
THEMES ?= $(patsubst %/index.theme,%,$(wildcard ./*/index.theme))
PKGNAME = flat-remix-gtk
MAINTAINER = Daniel Ruiz de Alegría <daniel@drasite.com>

# excludes IGNORE from THEMES list
THEMES := $(filter-out $(IGNORE), $(THEMES))

all:

build:
	cd src && ./build.sh

build-sass:
	cd src && ./build.sh --no-assets

install:
	mkdir -p $(DESTDIR)$(PREFIX)/share/themes
	cp -a $(THEMES) $(DESTDIR)$(PREFIX)/share/themes

uninstall:
	-rm -rf $(foreach theme,$(THEMES),$(DESTDIR)$(PREFIX)/share/themes/$(theme))

_get_version:
	$(eval VERSION ?= $(shell git show -s --format=%cd --date=format:%Y%m%d HEAD))
	@echo $(VERSION)

_get_tag:
	$(eval TAG := $(shell git describe --abbrev=0 --tags))
	@echo $(TAG)

dist: _get_version
	color_variants="-Blue -Green -Red -Yellow"; \
	theme_variants="- -Darker -Dark -Darkest"; \
	extra_variants="- -Solid -NoBorder -Solid-NoBorder"; \
	for color_variant in $$color_variants; \
	do \
		count=1; \
		for theme_variant in $$theme_variants; \
		do \
			[ "$$theme_variant" = '-' ] && theme_variant=''; \
			for extra_variant in $$extra_variants; \
			do \
				[ "$$extra_variant" = '-' ] && extra_variant=''; \
				file="Flat-Remix-GTK$${color_variant}$${theme_variant}$${extra_variant}"; \
				if [ -d "$$file" ]; \
				then \
					count_pretty=$$(echo "0$${count}" | tail -c 3); \
					tar -c "$$file" | \
							xz -z - > "$${count_pretty}-$${file}_$(VERSION).tar.xz"; \
					count=$$((count+1)); \
				fi; \
			done; \
		done; \
	done; \

release: _get_version
	$(MAKE) generate_changelog VERSION=$(VERSION)
	$(MAKE) aur_release VERSION=$(VERSION)
	$(MAKE) copr_release VERSION=$(VERSION)
	$(MAKE) launchpad_release
	git tag -f $(VERSION)
	git push origin --tags
	$(MAKE) dist

aur_release: _get_version _get_tag
	cd aur; \
	sed "s/$(TAG)/$(VERSION)/g" -i PKGBUILD .SRCINFO; \
	git commit -a -m "$(VERSION)"; \
	git push origin master;

	git commit aur -m "Update aur version $(VERSION)"
	git push origin master

copr_release: _get_version _get_tag
	sed "s/$(TAG)/$(VERSION)/g" -i $(PKGNAME).spec
	git commit $(PKGNAME).spec -m "Update $(PKGNAME).spec version $(VERSION)"
	git push origin master

launchpad_release: _get_version
	cp -a Flat-Remix* Makefile deb/$(PKGNAME)
	sed "s/{}/$(VERSION)/g" -i deb/$(PKGNAME)/debian/changelog-template
	cd deb/$(PKGNAME)/debian/ && echo " -- $(MAINTAINER)  $$(date -R)" | cat changelog-template - > changelog
	cd deb/$(PKGNAME) && debuild -S -d
	dput ppa deb/$(PKGNAME)_$(VERSION)_source.changes
	git checkout deb
	git clean -df deb

undo_release: _get_tag
	-git tag -d $(TAG)
	-git push --delete origin $(TAG)

generate_changelog: _get_version _get_tag
	git checkout $(TAG) CHANGELOG
	echo [$(VERSION)] > /tmp/out
	git log --pretty=format:' * %s' $(TAG)..HEAD >> /tmp/out
	echo >> /tmp/out
	echo | cat - CHANGELOG >> /tmp/out
	mv /tmp/out CHANGELOG
	$$EDITOR CHANGELOG
	git commit CHANGELOG -m "Update CHANGELOG version $(VERSION)"
	git push origin master


.PHONY: all build build-sass install uninstall _get_version _get_tag dist release aur_release copr_release launchpad_release undo_release generate_changelog

# .BEGIN is ignored by GNU make so we can use it as a guard
.BEGIN:
	@head -3 Makefile
	@false
