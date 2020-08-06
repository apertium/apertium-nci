###############################################################################
## Makefile for apertium-nci
###############################################################################

LANG1=nci
BASENAME=apertium-$(LANG1)

TARGETS_COMMON =			\
	$(LANG1).automorf.hfst		\
	$(LANG1).automorf.bin		\
	$(LANG1).automorf.att.gz	\
	$(LANG1).autogen.hfst		\
	$(LANG1).autophon.hfst		\
	$(LANG1).autogen.bin		\
	$(LANG1).autogen.att.gz		\
	$(LANG1).autoseg.hfst		\
	$(LANG1).autoseg.bin		\
	$(LANG1).prob		\
	$(LANG1).rlx.bin

TARGETS_INTERMEDIATE = $(LANG1).mor.hfst $(LANG1).seg.hfst $(LANG1).gen.hfst $(LANG1).phon.hfst

# This include defines goals for install-modes, .deps/.d and .mode files:
@ap_include@

###############################################################################

# nimahtlapaltzin:<s_sg1>mahtlapal<n><dim>
.deps/nci.mor.hfst: .deps/nci.gen.hfst .deps/nci.mor.twol.hfst
	hfst-compose-intersect -1 .deps/nci.gen.hfst -2 .deps/nci.mor.twol.hfst | hfst-minimise | hfst-invert -o $@

# amatl<n><dim><pl>:ama>tzi>tzin
.deps/nci.gen.hfst: .deps/nci.lexc.hfst .deps/nci.twol.hfst
	hfst-compose-intersect -1 .deps/nci.lexc.hfst -2 .deps/nci.twol.hfst | hfst-minimise  -o $@

.deps/nci.twol.hfst: apertium-nci.nci.twol
	hfst-twolc $< -o $@

# <px2sg>metztli<n><pl>:m{o}>metz>huan
.deps/nci.lexc.hfst: apertium-nci.nci.lexc .deps/.d
	hfst-lexc -A --Werror $< -o $@

.deps/nci.mor.twol.hfst: apertium-nci.nci.mor.twol
	hfst-twolc apertium-nci.nci.mor.twol -o .deps/nci.mor.twol.hfst

# notipehuan:no>tipe>huan
.deps/nci.seg.hfst: .deps/nci.mor.hfst .deps/nci.gen.hfst
	hfst-compose -1 .deps/nci.mor.hfst -2 .deps/nci.gen.hfst -o .deps/nci.seg.hfst 

.deps/nci.phon.hfst: apertium-nci.nci.phon
	hfst-lexc apertium-nci.nci.phon -o $@

.deps/nci.spellrelax.hfst: apertium-nci.nci.spellrelax 
	hfst-regexp2fst -S -o $@ < $<

$(LANG1).autophon.hfst: .deps/nci.phon.hfst
	hfst-fst2fst -w $< -o $@

$(LANG1).automorf.hfst: .deps/nci.mor.hfst .deps/nci.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 .deps/nci.spellrelax.hfst  | hfst-invert | hfst-fst2fst -w -o $@

$(LANG1).autogen.hfst: .deps/nci.gen.hfst
	hfst-fst2fst -w $< -o $@

$(LANG1).autoseg.hfst: .deps/nci.seg.hfst 
	hfst-fst2fst -w $< -o $@

$(LANG1).rlx.bin: apertium-nci.nci.rlx
	cg-comp $< $@

$(LANG1).autogen.att.gz: .deps/$(LANG1).gen.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).automorf.att.gz: .deps/$(LANG1).mor.hfst .deps/nci.spellrelax.hfst
	hfst-invert $< |  hfst-compose -F -1 - -2 .deps/nci.spellrelax.hfst  | hfst-invert | hfst-fst2txt | gzip -9 -c > $@

$(LANG1).autoseg.att.gz: .deps/$(LANG1).seg.hfst
	hfst-fst2txt $< | gzip -9 -c > $@

$(LANG1).autogen.bin: $(LANG1).autogen.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autogen.att
	lt-comp lr .deps/$(LANG1).autogen.att $@

$(LANG1).automorf.bin: $(LANG1).automorf.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).automorf.att
	lt-comp lr .deps/$(LANG1).automorf.att $@

$(LANG1).autoseg.bin: $(LANG1).autoseg.att.gz .deps/.d
	zcat < $< > .deps/$(LANG1).autoseg.att
	lt-comp lr .deps/$(LANG1).autoseg.att $@

$(LANG1).prob: $(BASENAME).$(LANG1).tagged $(LANG1).rlx.bin
	cat $< | cg-proc $(LANG1).rlx.bin | apertium-tagger -s 0 -u 2 $@ /dev/stdin

###############################################################################
## Distribution
###############################################################################

EXTRA_DIST=$(BASENAME).$(LANG1).lexc		\
		$(BASENAME).$(LANG1).twol	\
		$(BASENAME).$(LANG1).phon \
		$(BASENAME).$(LANG1).spellrelax \
		$(BASENAME).$(LANG1).rlx	\
		modes.xml

###############################################################################
## Installation stuff
###############################################################################
#
#   apertium_nci_dir: This is where the compiled binaries go
#   apertium_nci_srcdir: This is where the source files go

apertium_ncidir=$(prefix)/share/apertium/$(BASENAME)/
apertium_nci_srcdir=$(prefix)/share/apertium/$(BASENAME)/

EXTRA_TARGETS=

if HAVE_HFSTOSPELL
EXTRA_TARGETS += $(LANG1).zhfst
endif # HAVE_HFSTOSPELL

apertium_nci_DATA=$(TARGETS_COMMON) $(EXTRA_TARGETS) 

pkgconfigdir = $(prefix)/share/pkgconfig
pkgconfig_DATA = $(BASENAME).pc

noinst_DATA=modes/$(LANG1)-morph.mode

install-data-local: install-modes
	$(INSTALL_DATA) $(BASENAME).$(LANG1).lexc $(DESTDIR)$(apertium_nci_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).twol $(DESTDIR)$(apertium_nci_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).rlx $(DESTDIR)$(apertium_nci_srcdir)


###############################################################################
## Cleanup
###############################################################################

CLEANFILES = $(TARGETS_COMMON) $(TARGETS_INTERMEDIATE)
clean-local:
	-rm -rf .deps modes

###############################################################################
