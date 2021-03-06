BASEDIR=../..

CURRENT_MAKEFILE=$(MAKEFILE_LIST)
GIT_CHECK=$(shell { [ -f $(BASEDIR)/.git/config ] && [ -x "$$(which git)" ]; } || echo git)

# Configuration section
ifneq ($(wildcard config.mk),)
include config.mk
endif
INSTALLROOT?=
PREFIX?=$(INSTALLROOT)/usr/local
BIN?=$(PREFIX)/bin
DOC?=$(PREFIX)/share/doc/gws
BASH_COMPLETION?=$(INSTALLROOT)/etc/bash_completion.d
ZSH_COMPLETION?=$(INSTALLROOT)/usr/share/zsh/vendor-completions
ifeq (,$(GIT_CHECK))
MAINTAINER?=$(shell git log -1 --format='%an <%ae>' -- $(CURRENT_MAKEFILE))
endif

DIRECTORIES=$(BIN) $(DOC) $(BASH_COMPLETION) $(ZSH_COMPLETION)
INSTALL=$(BIN)/gws $(DOC)/changelog.gz $(DOC)/license $(DOC)/readme.md.gz $(BASH_COMPLETION)/gws $(ZSH_COMPLETION)/_gws
DEBIAN_TOOLS=dpkg dpkg-deb fakeroot
DEBIAN_FILES=$(shell ( cd debian; find . -type f ))
DEBIAN_TOOLS_CHECK=$(shell for tool in $(DEBIAN_TOOLS); do which $$tool 2>&1 > /dev/null || echo $$tool; done )
ifeq (,$(GIT_CHECK))
VERSION=$(shell git describe --dirty --tags --match '[0-9]*\.[0-9]*\.[0-9]*')
endif
PACKAGE=gws_$(VERSION)

help:
	$(info Available targets are:)
	$(info $(tab)- test:                prepares a workspace for manual tests)
	$(info $(tab)- install (as root):   install gws with $(PREFIX) preffix)
	$(info $(tab)- uninstall (as root): uninstall gws with $(PREFIX) preffix)
ifeq (,$(DEBIAN_TOOLS_CHECK))
	$(info $(tab)- debian:              build a Debian package)
endif
	$(info $(tab)- clean:               clean built stuff)

all:

$(INSTALL): pre.install

install: $(INSTALL)

pre.install:
ifeq (,$(GIT_CHECK))
	$(if $(shell git diff-index --quiet HEAD || echo dirty),\
		$(warning Git repository is dirty) \
	)
else
ifeq (,$(VERSION))
	$(error Please define VERSION)
endif
endif
	@echo Install version: $(VERSION)

uninstall:
	rm -f $(INSTALL)
	rmdir $(DOC)

ifeq (,$(DEBIAN_TOOLS_CHECK))
debian: pre.package.check $(PACKAGE)_all.deb
endif

test:
	echo "BASEDIR: $(BASEDIR)"
	$(MAKE) --directory=$(BASEDIR)/tests tests

clean:
	$(MAKE) --directory=$(BASEDIR)/tests clean
	rm -rf gws_*

$(DIRECTORIES):
	@[ -d $@ ] || mkdir --parent $@

$(BIN)/gws: $(BASEDIR)/src/gws $(BIN)
	cp $< $@

$(DOC)/changelog.gz: $(BASEDIR)/CHANGELOG $(DOC)
	gzip -c $< > $@

$(DOC)/license: $(BASEDIR)/LICENSE $(DOC)
	cp $< $@

$(DOC)/readme.md.gz: $(BASEDIR)/README.md $(DOC)
	gzip -c $< > $@

$(ZSH_COMPLETION)/_gws: $(BASEDIR)/completions/zsh $(ZSH_COMPLETION)
	cp $< $@

$(BASH_COMPLETION)/gws: $(BASEDIR)/completions/bash $(BASH_COMPLETION)
	cp $< $@

$(PACKAGE)/DEBIAN/./control: debian/$(DEBIAN_FILES)
	[ -d $@ ] || mkdir --parent $(dir $@)
	fakeroot sed 's/@@version@@/$(VERSION)/; s/@@maintainer@@/$(MAINTAINER)/' $^ > $@

$(PACKAGE)_all.deb: $(addprefix $(PACKAGE)/DEBIAN/,$(DEBIAN_FILES))
	$(MAKE) install PREFIX=$(PACKAGE)/usr BASH_COMPLETION=$(PACKAGE)/$(BASH_COMPLETION) ZSH_COMPLETION=$(PACKAGE)/$(ZSH_COMPLETION)
	fakeroot dpkg --build $(PACKAGE)

pre.package.check:
ifeq (,$(MAINTAINER))
	$(error Please define MAINTAINER)
endif

define tab
	
endef

.PHONY: help all test install uninstall debian pre.install pre.package.check
