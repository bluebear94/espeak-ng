AUTOMAKE_OPTIONS = subdir-objects
ACLOCAL_AMFLAGS = -I m4

PREFIX=@prefix@
BINDIR=@bindir@
INCDIR=@includedir@/espeak
LIBDIR=@libdir@
DATADIR=$(PREFIX)/share/espeak-ng-data
VIMDIR=$(PREFIX)/share/vim

pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = espeak-ng.pc

SHARED_VERSION=2:49:1 # = *.so.1.1.49

MKDIR=mkdir -p

# -Wno-endif-labels : Needed to prevent warnings in ieee80.c.
AM_CFLAGS = \
	-Isrc/include -Isrc/include/compat -I src/ucd-tools/src/include \
	-D_BSD_SOURCE -D_DEFAULT_SOURCE -D_POSIX_C_SOURCE=200112L \
	-Wno-endif-labels

EXTRA_DIST=

bin_PROGRAMS =
lib_LTLIBRARIES =
man1_MANS =

check_PROGRAMS =
check_LTLIBRARIES =

##### ChangeLog:

ChangeLog:
	git log > ChangeLog

dist-hook: ChangeLog

.PHONY: ChangeLog

EXTRA_DIST += ChangeLog

##### standard build actions:

all-local: \
	espeak-ng-data/phontab \
	dictionaries \
	mbrola

uninstall-hook:
	rm -rf $(DESTDIR)$(DATADIR)

install-data-hook: uninstall-hook
	$(MKDIR) $(DESTDIR)$(DATADIR)
	cp -prf espeak-ng-data/* $(DESTDIR)$(DATADIR)

install-exec-hook:
	cd $(DESTDIR)$(BINDIR) && rm -f espeak && $(LN_S) espeak-ng espeak
	cd $(DESTDIR)$(BINDIR) && rm -f speak  && $(LN_S) speak-ng speak
	cd $(DESTDIR)$(LIBDIR) && rm -f libespeak.la && $(LN_S) libespeak-ng.la libespeak.la

clean-local:
	cd src && rm -f *.o *~ && cd ..
	cd src/libespeak-ng/ && rm -f *.o *.lo && cd ../..
	cd tests && rm -f *.o *.lo && cd ..
if HAVE_GRADLE
	cd android && $(GRADLE) clean
endif

distclean-local:
	rm -rf espeak-ng-data/phondata-manifest
	rm -f espeak-ng-data/*_dict

##### custom rules:

SUFFIXES=.html .md .ronn .check .test

.md.html: _layouts/webpage.html
	@echo "  MD        $@"
	@cat $< | sed -e 's/\.md)/.html)/g' -e 's/\.ronn/.html/g' | \
		kramdown --template _layouts/webpage.html > $@

.ronn.html:
	ronn --html $<

##### vim:

vim_registrydir = $(VIMDIR)/registry
vim_registry_DATA = vim/registry/*.yaml
EXTRA_DIST += $(vim_registry_DATA)

vim_addons_ftdetectdir = $(VIMDIR)/addons/ftdetect
vim_addons_ftdetect_DATA = vim/ftdetect/*.vim
EXTRA_DIST += $(vim_addons_ftdetect_DATA)

vim_addons_syntaxdir = $(VIMDIR)/addons/syntax
vim_addons_syntax_DATA = vim/syntax/*.vim
EXTRA_DIST += $(vim_addons_syntax_DATA)

##### documentation:

src/espeak-ng.1: src/espeak-ng.1.ronn
	ronn --roff $<

src/speak-ng.1: src/speak-ng.1.ronn
	ronn --roff $<

docs_MARKDOWN != ls docs/*.md docs/*/*.md docs/*/*/*.md
docs_HTML = ${docs_MARKDOWN:.md=.html}

docs:	$(docs_HTML) \
	src/espeak-ng.1.html \
	src/speak-ng.1.html \
	CHANGELOG.html \
	README.html

##### build targets:

espeak_includedir = $(includedir)/espeak
espeak_include_HEADERS = \
	src/include/espeak/speak_lib.h

espeak_ng_includedir = $(includedir)/espeak-ng
espeak_ng_include_HEADERS = \
	src/include/espeak-ng/encoding.h \
	src/include/espeak-ng/espeak_ng.h \
	src/include/espeak-ng/speak_lib.h

lib_LTLIBRARIES += src/libespeak-ng.la

src_libespeak_ng_la_LDFLAGS = -version-info $(SHARED_VERSION) -lpthread -lm \
	${PCAUDIOLIB_LIBS}

src_libespeak_ng_la_CFLAGS = \
	-fPIC -fvisibility=hidden \
	-pedantic -fno-exceptions -D PATH_ESPEAK_DATA=\"$(DATADIR)\" -DLIBESPEAK_NG_EXPORT \
	${PCAUDIOLIB_CFLAGS} ${AM_CFLAGS}

src_libespeak_ng_la_SOURCES = \
	src/ucd-tools/src/case.c \
	src/ucd-tools/src/categories.c \
	src/ucd-tools/src/ctype.c \
	src/ucd-tools/src/proplist.c \
	src/ucd-tools/src/scripts.c \
	src/ucd-tools/src/tostring.c \
	src/libespeak-ng/compiledata.c \
	src/libespeak-ng/compiledict.c \
	src/libespeak-ng/compilembrola.c \
	src/libespeak-ng/dictionary.c \
	src/libespeak-ng/encoding.c \
	src/libespeak-ng/error.c \
	src/libespeak-ng/espeak_api.c \
	src/libespeak-ng/ieee80.c \
	src/libespeak-ng/intonation.c \
	src/libespeak-ng/mnemonics.c \
	src/libespeak-ng/numbers.c \
	src/libespeak-ng/readclause.c \
	src/libespeak-ng/phoneme.c \
	src/libespeak-ng/phonemelist.c \
	src/libespeak-ng/setlengths.c \
	src/libespeak-ng/spect.c \
	src/libespeak-ng/speech.c \
	src/libespeak-ng/ssml.c \
	src/libespeak-ng/synthdata.c \
	src/libespeak-ng/synthesize.c \
	src/libespeak-ng/synth_mbrola.c \
	src/libespeak-ng/translate.c \
	src/libespeak-ng/tr_languages.c \
	src/libespeak-ng/voices.c \
	src/libespeak-ng/wavegen.c

if OPT_KLATT
src_libespeak_ng_la_CFLAGS  += -DINCLUDE_KLATT
src_libespeak_ng_la_SOURCES += src/libespeak-ng/klatt.c
endif

if OPT_MBROLA
src_libespeak_ng_la_CFLAGS  += -DINCLUDE_MBROLA
src_libespeak_ng_la_SOURCES += src/libespeak-ng/mbrowrap.c
endif

if OPT_ASYNC
src_libespeak_ng_la_CFLAGS  += -DUSE_ASYNC
src_libespeak_ng_la_SOURCES += \
	src/libespeak-ng/espeak_command.c \
	src/libespeak-ng/event.c \
	src/libespeak-ng/fifo.c
endif

bin_PROGRAMS += src/speak-ng

if HAVE_RONN
man1_MANS += src/speak-ng.1
endif

src_speak_ng_LDADD   = src/libespeak-ng.la
src_speak_ng_LDFLAGS = -static -lm ${PCAUDIOLIB_LIBS}
src_speak_ng_CFLAGS  = -Isrc/libespeak-ng ${AM_CFLAGS}
src_speak_ng_SOURCES = src/speak-ng.c

bin_PROGRAMS += src/espeak-ng

if HAVE_RONN
man1_MANS += src/espeak-ng.1
endif

src_espeak_ng_LDADD   = src/libespeak-ng.la ${PCAUDIOLIB_LIBS}
src_espeak_ng_SOURCES = src/espeak-ng.c

##### tests:

# Test version of libespeak-ng.so with access to the internal APIs, so they can
# be accessed in the test code. This version should not be installed, as the
# internal APIs are not guaranteed to be stable between releases.
check_LTLIBRARIES += src/libespeak-ng-test.la

src_libespeak_ng_test_la_LDFLAGS = $(src_libespeak_ng_la_LDFLAGS)
src_libespeak_ng_test_la_CFLAGS  = \
	-fPIC -pedantic -fno-exceptions -D PATH_ESPEAK_DATA=\"$(DATADIR)\" -DLIBESPEAK_NG_EXPORT \
	${PCAUDIOLIB_CFLAGS} ${AM_CFLAGS}
src_libespeak_ng_test_la_SOURCES = $(src_libespeak_ng_la_SOURCES)

check_PROGRAMS += tests/encoding.test

tests_encoding_test_LDADD   = src/libespeak-ng.la
tests_encoding_test_SOURCES = tests/encoding.c

check_PROGRAMS += tests/readclause.test

tests_readclause_test_CFLAGS  = -Isrc/libespeak-ng ${AM_CFLAGS}
tests_readclause_test_LDADD   = src/libespeak-ng-test.la
tests_readclause_test_SOURCES = tests/readclause.c

check_PROGRAMS += tests/api.test

tests_api_test_CFLAGS  = -Isrc/libespeak-ng ${AM_CFLAGS}
tests_api_test_LDADD   = src/libespeak-ng-test.la
tests_api_test_SOURCES = tests/api.c

.test.check:
	@echo "  TEST      $<"
	@ESPEAK_DATA_PATH=$(PWD) $< && echo "  PASSED    $<"

check:	tests/encoding.check \
	tests/readclause.check \
	tests/translate.check \
	tests/ssml.check \
	tests/ssml-fuzzer.check \
	tests/api.check \
	tests/language-phonemes.check \
	tests/language-replace.check \
	tests/language-pronunciation.check \
	tests/language-numbers-cardinal.check \
	tests/language-numbers-ordinal.check

##### fuzzer:

if !HAVE_LIBFUZZER
# libfuzzrunner is a stub implementation of libFuzzer that calls the
# LLVMFuzzerTestOneInput with the files passed to the fuzzer program.
check_LTLIBRARIES += tests/libfuzzrunner.la
tests_libfuzzrunner_la_CFLAGS  = -Isrc/libespeak-ng ${AM_CFLAGS}
tests_libfuzzrunner_la_SOURCES = tests/fuzzrunner.c
endif

check_PROGRAMS += tests/ssml-fuzzer.test
tests_ssml_fuzzer_test_CFLAGS  = ${AM_CFLAGS}
tests_ssml_fuzzer_test_SOURCES = tests/ssml-fuzzer.c
tests_ssml_fuzzer_test_LDADD   = src/libespeak-ng.la

if HAVE_LIBFUZZER
tests_ssml_fuzzer_test_CFLAGS += -fsanitize=fuzzer
tests_ssml_fuzzer_test_LDFLAGS = -fsanitize=fuzzer
else
tests_ssml_fuzzer_test_LDADD += tests/libfuzzrunner.la
endif

tests/ssml-fuzzer.check: tests/ssml-fuzzer.test
	@echo "  TEST      $<"
	@ESPEAK_DATA_PATH=$(PWD) $< tests/ssml/*.ssml tests/ssml-fuzzer/*.ssml && echo "  PASSED    $<"


##### phoneme data:

espeak-ng-data/phondata: phsource/phonemes.stamp
espeak-ng-data/phondata-manifest: phsource/phonemes.stamp
espeak-ng-data/phonindex: phsource/phonemes.stamp
espeak-ng-data/phontab: phsource/phonemes.stamp
espeak-ng-data/intonations: phsource/phonemes.stamp

phsource/phonemes.stamp: \
	phsource/ph_afrikaans \
	phsource/ph_akan \
	phsource/ph_albanian \
	phsource/ph_amhari \
	phsource/ph_arabic \
	phsource/ph_aragon \
	phsource/ph_arka \
	phsource/ph_armenian \
	phsource/ph_assamese \
	phsource/ph_azerbaijani \
	phsource/ph_base2 \
	phsource/ph_basque \
	phsource/ph_bengali \
	phsource/ph_bulgarian \
	phsource/ph_myanmar \
	phsource/ph_catalan \
	phsource/ph_consonants \
	phsource/ph_croatian \
	phsource/ph_czech \
	phsource/ph_danish \
	phsource/ph_dari \
	phsource/ph_divehi \
	phsource/ph_dutch \
	phsource/ph_english \
	phsource/ph_english_n \
	phsource/ph_english_rp \
	phsource/ph_english_sc \
	phsource/ph_english_us \
	phsource/ph_english_wi \
	phsource/ph_english_wm \
	phsource/ph_esperanto \
	phsource/ph_estonian \
	phsource/ph_farsi \
	phsource/ph_finnish \
	phsource/ph_french \
	phsource/ph_georgian \
	phsource/ph_german \
	phsource/ph_greek \
	phsource/ph_greek_ancient \
	phsource/ph_greenlandic \
	phsource/ph_gujarati \
	phsource/ph_haitian \
	phsource/ph_hakka \
	phsource/ph_hindi \
	phsource/ph_hindi_base \
	phsource/ph_hungarian \
	phsource/ph_icelandic \
	phsource/ph_indonesian \
	phsource/ph_interlingua \
	phsource/ph_irish \
	phsource/ph_italian \
	phsource/ph_japanese \
	phsource/ph_kannada \
	phsource/ph_kazakh \
	phsource/ph_kinyarwanda \
	phsource/ph_korean \
	phsource/ph_kurdish \
	phsource/ph_kyrgyz \
	phsource/ph_latin \
	phsource/ph_latvian \
	phsource/ph_lithuanian \
	phsource/ph_lojban \
	phsource/ph_macedonian \
	phsource/ph_malayalam \
	phsource/ph_maltese \
	phsource/ph_manipuri \
	phsource/ph_maori \
	phsource/ph_marathi \
	phsource/ph_nahuatl \
	phsource/ph_nepali \
	phsource/ph_northern-sotho \
	phsource/ph_norwegian \
	phsource/ph_oriya \
	phsource/ph_oromo \
	phsource/ph_polish \
	phsource/ph_portugal \
	phsource/ph_pt_brazil \
	phsource/ph_punjabi \
	phsource/ph_pyash \
	phsource/ph_romanian \
	phsource/ph_russian \
	phsource/ph_russian_lv \
	phsource/ph_s_gaelic \
	phsource/ph_serbian \
	phsource/ph_setswana \
	phsource/ph_shan \
	phsource/ph_sinhala \
	phsource/ph_slovak \
	phsource/ph_slovenian \
	phsource/ph_spanish \
	phsource/ph_spanish_la \
	phsource/ph_swahili \
	phsource/ph_swedish \
	phsource/ph_tamil \
	phsource/ph_tatar \
	phsource/ph_telugu \
	phsource/ph_tibetan \
	phsource/ph_turkish \
	phsource/ph_urdu \
	phsource/ph_vietnam \
	phsource/ph_vietnam_hue \
	phsource/ph_vietnam_sgn \
	phsource/ph_welsh \
	phsource/ph_wolof \
	phsource/ph_yoruba \
	phsource/ph_zh \
	phsource/ph_zh_yue \
	phsource/phonemes \
	src/espeak-ng
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-intonations && \
		ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-phonemes && \
		touch $@

##### android targets:

jni:
	cd android && ndk-build

apk-release: jni espeakdata
	cd android && $(GRADLE) assembleRelease

apk-debug: jni espeakdata
	cd android && $(GRADLE) assembleDebug

apk-check: jni espeakdata
	cd android && $(GRADLE) connectedCheck

android/res/raw/espeakdata.zip: espeak-ng-data/phontab dictionaries
	mkdir -pv android/res/raw
	rm -f $@
	find espeak-ng-data/{intonations,phondata,phonindex,phontab} | zip -@ $@
	find espeak-ng-data/*_dict | zip -@ $@
	find espeak-ng-data/lang -type f | zip -@ $@
	find espeak-ng-data/voices -type f | grep -vF "/mb/" | zip -@ $@

android/res/raw/espeakdata_version: android/res/raw/espeakdata.zip
	sha1sum $< | awk '{ print $$1 }' > $@

espeakdata: \
	android/res/raw/espeakdata.zip \
	android/res/raw/espeakdata_version

##### dictionaries:

dictsource/%_emoji:
	@echo "  EMOJI     $@"
	@if test x"${CLDR_PATH}" = x ; then \
		touch $@ ; \
	else \
		tools/emoji \
			dictsource/en_emoji \
			`echo $@ | sed -e 's,dictsource/,,g' -e 's,_emoji,,g'` \
			${CLDR_PATH} > $@ ; \
	fi

dictsource/%_extra:
	touch $@

espeak-ng-data/%_dict: src/espeak-ng phsource/phonemes.stamp
	@echo "  DICT      $@"
	@cd dictsource && ESPEAK_DATA_PATH=$(PWD) LD_LIBRARY_PATH=../src:${LD_LIBRARY_PATH} ../src/espeak-ng \
		--compile=`echo $@ | sed -e 's,espeak-ng-data/,,g' -e 's,_dict,,g'` && cd ..

dictionaries: \
	espeak-ng-data/af_dict \
	espeak-ng-data/am_dict \
	espeak-ng-data/an_dict \
	espeak-ng-data/ar_dict \
	espeak-ng-data/arka_dict \
	espeak-ng-data/as_dict \
	espeak-ng-data/az_dict \
	espeak-ng-data/bg_dict \
	espeak-ng-data/bn_dict \
	espeak-ng-data/bpy_dict \
	espeak-ng-data/bs_dict \
	espeak-ng-data/ca_dict \
	espeak-ng-data/cs_dict \
	espeak-ng-data/cy_dict \
	espeak-ng-data/da_dict \
	espeak-ng-data/de_dict \
	espeak-ng-data/el_dict \
	espeak-ng-data/en_dict \
	espeak-ng-data/eo_dict \
	espeak-ng-data/es_dict \
	espeak-ng-data/et_dict \
	espeak-ng-data/eu_dict \
	espeak-ng-data/fa_dict \
	espeak-ng-data/fi_dict \
	espeak-ng-data/fr_dict \
	espeak-ng-data/ga_dict \
	espeak-ng-data/gd_dict \
	espeak-ng-data/gn_dict \
	espeak-ng-data/grc_dict \
	espeak-ng-data/gu_dict \
	espeak-ng-data/hak_dict \
	espeak-ng-data/hi_dict \
	espeak-ng-data/hr_dict \
	espeak-ng-data/ht_dict \
	espeak-ng-data/hu_dict \
	espeak-ng-data/hy_dict \
	espeak-ng-data/ia_dict \
	espeak-ng-data/id_dict \
	espeak-ng-data/is_dict \
	espeak-ng-data/it_dict \
	espeak-ng-data/ja_dict \
	espeak-ng-data/jbo_dict \
	espeak-ng-data/ka_dict \
	espeak-ng-data/kk_dict \
	espeak-ng-data/kl_dict \
	espeak-ng-data/kn_dict \
	espeak-ng-data/ko_dict \
	espeak-ng-data/kok_dict \
	espeak-ng-data/ku_dict \
	espeak-ng-data/ky_dict \
	espeak-ng-data/la_dict \
	espeak-ng-data/lfn_dict \
	espeak-ng-data/lt_dict \
	espeak-ng-data/lv_dict \
	espeak-ng-data/mi_dict \
	espeak-ng-data/mk_dict \
	espeak-ng-data/ml_dict \
	espeak-ng-data/mr_dict \
	espeak-ng-data/ms_dict \
	espeak-ng-data/mt_dict \
	espeak-ng-data/my_dict \
	espeak-ng-data/nci_dict \
	espeak-ng-data/ne_dict \
	espeak-ng-data/nl_dict \
	espeak-ng-data/no_dict \
	espeak-ng-data/om_dict \
	espeak-ng-data/or_dict \
	espeak-ng-data/pa_dict \
	espeak-ng-data/pap_dict \
	espeak-ng-data/pl_dict \
	espeak-ng-data/pt_dict \
	espeak-ng-data/py_dict \
	espeak-ng-data/quc_dict \
	espeak-ng-data/ro_dict \
	espeak-ng-data/ru_dict \
	espeak-ng-data/sd_dict \
	espeak-ng-data/shn_dict \
	espeak-ng-data/si_dict \
	espeak-ng-data/sk_dict \
	espeak-ng-data/sl_dict \
	espeak-ng-data/sq_dict \
	espeak-ng-data/sr_dict \
	espeak-ng-data/sv_dict \
	espeak-ng-data/sw_dict \
	espeak-ng-data/ta_dict \
	espeak-ng-data/te_dict \
	espeak-ng-data/tn_dict \
	espeak-ng-data/tr_dict \
	espeak-ng-data/tt_dict \
	espeak-ng-data/ur_dict \
	espeak-ng-data/vi_dict \
	espeak-ng-data/zh_dict \
	espeak-ng-data/zhy_dict

af: espeak-ng-data/af_dict
espeak-ng-data/af_dict: dictsource/af_list dictsource/af_rules dictsource/af_extra dictsource/af_emoji

am: espeak-ng-data/am_dict
espeak-ng-data/am_dict: dictsource/am_list dictsource/am_rules dictsource/am_extra dictsource/am_emoji

an: espeak-ng-data/an_dict
espeak-ng-data/an_dict: dictsource/an_list dictsource/an_rules dictsource/an_extra

ar: espeak-ng-data/ar_dict
espeak-ng-data/ar_dict: dictsource/ar_list dictsource/ar_listx dictsource/ar_rules dictsource/ar_extra dictsource/ar_emoji

arka: espeak-ng-data/arka_dict
espeak-ng-data/arka_dict: dictsource/arka_list dictsource/arka_rules

as: espeak-ng-data/as_dict
espeak-ng-data/as_dict: dictsource/as_list dictsource/as_rules dictsource/as_extra

az: espeak-ng-data/az_dict
espeak-ng-data/az_dict: dictsource/az_list dictsource/az_rules dictsource/az_extra dictsource/az_emoji

bg: espeak-ng-data/bg_dict
espeak-ng-data/bg_dict: dictsource/bg_list dictsource/bg_listx dictsource/bg_rules dictsource/bg_extra dictsource/bg_emoji

bn: espeak-ng-data/bn_dict
espeak-ng-data/bn_dict: dictsource/bn_list dictsource/bn_rules dictsource/bn_extra dictsource/bn_emoji

bpy: espeak-ng-data/bpy_dict
espeak-ng-data/bpy_dict: dictsource/bpy_list dictsource/bpy_rules dictsource/bpy_extra

bs: espeak-ng-data/bs_dict
espeak-ng-data/bs_dict: dictsource/bs_list dictsource/bs_rules dictsource/bs_extra dictsource/bs_emoji

ca: espeak-ng-data/ca_dict
espeak-ng-data/ca_dict: dictsource/ca_list dictsource/ca_rules dictsource/ca_extra dictsource/ca_emoji

cs: espeak-ng-data/cs_dict
espeak-ng-data/cs_dict: dictsource/cs_list dictsource/cs_rules dictsource/cs_extra dictsource/cs_emoji

cy: espeak-ng-data/cy_dict
espeak-ng-data/cy_dict: dictsource/cy_list dictsource/cy_rules dictsource/cy_extra dictsource/cy_emoji

da: espeak-ng-data/da_dict
espeak-ng-data/da_dict: dictsource/da_list dictsource/da_rules dictsource/da_extra dictsource/da_emoji

de: espeak-ng-data/de_dict
espeak-ng-data/de_dict: dictsource/de_list dictsource/de_rules dictsource/de_extra dictsource/de_emoji

el: espeak-ng-data/el_dict
espeak-ng-data/el_dict: dictsource/el_list dictsource/el_rules dictsource/el_extra dictsource/el_emoji

en: espeak-ng-data/en_dict
espeak-ng-data/en_dict: dictsource/en_list dictsource/en_rules dictsource/en_extra dictsource/en_emoji

eo: espeak-ng-data/eo_dict
espeak-ng-data/eo_dict: dictsource/eo_list dictsource/eo_rules dictsource/eo_extra

es: espeak-ng-data/es_dict
espeak-ng-data/es_dict: dictsource/es_list dictsource/es_rules dictsource/es_extra dictsource/es_emoji

et: espeak-ng-data/et_dict
espeak-ng-data/et_dict: dictsource/et_list dictsource/et_rules dictsource/et_extra dictsource/et_emoji

eu: espeak-ng-data/eu_dict
espeak-ng-data/eu_dict: dictsource/eu_list dictsource/eu_rules dictsource/eu_extra dictsource/eu_emoji

fa: espeak-ng-data/fa_dict
espeak-ng-data/fa_dict: dictsource/fa_list dictsource/fa_rules dictsource/fa_extra dictsource/fa_emoji

fi: espeak-ng-data/fi_dict
espeak-ng-data/fi_dict: dictsource/fi_list dictsource/fi_rules dictsource/fi_extra dictsource/fi_emoji

fr: espeak-ng-data/fr_dict
espeak-ng-data/fr_dict: dictsource/fr_list dictsource/fr_rules dictsource/fr_extra dictsource/fr_emoji

hak: espeak-ng-data/hak_dict
espeak-ng-data/hak_dict: dictsource/hak_list dictsource/hak_rules dictsource/hak_extra

ht: espeak-ng-data/ht_dict
espeak-ng-data/ht_dict: dictsource/ht_list dictsource/ht_rules dictsource/ht_extra

ga: espeak-ng-data/ga_dict
espeak-ng-data/ga_dict: dictsource/ga_list dictsource/ga_rules dictsource/ga_extra dictsource/ga_emoji

gd: espeak-ng-data/gd_dict
espeak-ng-data/gd_dict: dictsource/gd_list dictsource/gd_rules dictsource/gd_extra dictsource/gd_emoji

gn: espeak-ng-data/gn_dict
espeak-ng-data/gn_dict: dictsource/gn_list dictsource/gn_rules dictsource/gn_extra

grc: espeak-ng-data/grc_dict
espeak-ng-data/grc_dict: dictsource/grc_list dictsource/grc_rules dictsource/grc_extra

gu: espeak-ng-data/gu_dict
espeak-ng-data/gu_dict: dictsource/gu_list dictsource/gu_rules dictsource/gu_extra dictsource/gu_emoji

hi: espeak-ng-data/hi_dict
espeak-ng-data/hi_dict: dictsource/hi_list dictsource/hi_rules dictsource/hi_extra dictsource/hi_emoji

hr: espeak-ng-data/hr_dict
espeak-ng-data/hr_dict: dictsource/hr_list dictsource/hr_rules dictsource/hr_extra dictsource/hr_emoji

hu: espeak-ng-data/hu_dict
espeak-ng-data/hu_dict: dictsource/hu_list dictsource/hu_rules dictsource/hu_extra dictsource/hu_emoji

hy: espeak-ng-data/hy_dict
espeak-ng-data/hy_dict: dictsource/hy_list dictsource/hy_rules dictsource/hy_extra dictsource/hy_emoji

ia: espeak-ng-data/ia_dict
espeak-ng-data/ia_dict: dictsource/ia_list dictsource/ia_rules dictsource/ia_extra

id: espeak-ng-data/id_dict
espeak-ng-data/id_dict: dictsource/id_list dictsource/id_rules dictsource/id_extra dictsource/id_emoji

is: espeak-ng-data/is_dict
espeak-ng-data/is_dict: dictsource/is_list dictsource/is_rules dictsource/is_extra dictsource/is_emoji

it: espeak-ng-data/it_dict
espeak-ng-data/it_dict: dictsource/it_list dictsource/it_listx dictsource/it_rules dictsource/it_extra dictsource/it_emoji

jbo: espeak-ng-data/jbo_dict
espeak-ng-data/jbo_dict: dictsource/jbo_list dictsource/jbo_rules dictsource/jbo_extra

ja: espeak-ng-data/ja_dict
espeak-ng-data/ja_dict: dictsource/ja_list dictsource/ja_rules dictsource/ja_extra dictsource/ja_emoji

ka: espeak-ng-data/ka_dict
espeak-ng-data/ka_dict: dictsource/ka_list dictsource/ka_rules dictsource/ka_extra dictsource/ka_emoji

kk: espeak-ng-data/kk_dict
espeak-ng-data/kk_dict: dictsource/kk_list dictsource/kk_rules dictsource/kk_extra dictsource/kk_emoji

kl: espeak-ng-data/kl_dict
espeak-ng-data/kl_dict: dictsource/kl_list dictsource/kl_rules dictsource/kl_extra

kn: espeak-ng-data/kn_dict
espeak-ng-data/kn_dict: dictsource/kn_list dictsource/kn_rules dictsource/kn_extra dictsource/kn_emoji

ko: espeak-ng-data/ko_dict
espeak-ng-data/ko_dict: dictsource/ko_list dictsource/ko_rules dictsource/ko_extra dictsource/ko_emoji

kok: espeak-ng-data/kok_dict
espeak-ng-data/kok_dict: dictsource/kok_list dictsource/kok_rules dictsource/kok_extra

ku: espeak-ng-data/ku_dict
espeak-ng-data/ku_dict: dictsource/ku_list dictsource/ku_rules dictsource/ku_extra

ky: espeak-ng-data/ky_dict
espeak-ng-data/ky_dict: dictsource/ky_list dictsource/ky_rules dictsource/ky_extra dictsource/ky_emoji

la: espeak-ng-data/la_dict
espeak-ng-data/la_dict: dictsource/la_list dictsource/la_rules dictsource/la_extra

lfn: espeak-ng-data/lfn_dict
espeak-ng-data/lfn_dict: dictsource/lfn_list dictsource/lfn_rules dictsource/lfn_extra

lt: espeak-ng-data/lt_dict
espeak-ng-data/lt_dict: dictsource/lt_list dictsource/lt_rules dictsource/lt_extra dictsource/lt_emoji

lv: espeak-ng-data/lv_dict
espeak-ng-data/lv_dict: dictsource/lv_list dictsource/lv_rules dictsource/lv_extra dictsource/lv_emoji

mi: espeak-ng-data/mi_dict
espeak-ng-data/mi_dict: dictsource/mi_list dictsource/mi_rules dictsource/mi_extra

mk: espeak-ng-data/mk_dict
espeak-ng-data/mk_dict: dictsource/mk_list dictsource/mk_rules dictsource/mk_extra dictsource/mk_emoji

ml: espeak-ng-data/ml_dict
espeak-ng-data/ml_dict: dictsource/ml_list dictsource/ml_rules dictsource/ml_extra dictsource/ml_emoji

mr: espeak-ng-data/mr_dict
espeak-ng-data/mr_dict: dictsource/mr_list dictsource/mr_rules dictsource/mr_extra dictsource/mr_emoji

ms: espeak-ng-data/ms_dict
espeak-ng-data/ms_dict: dictsource/ms_list dictsource/ms_rules dictsource/ms_extra dictsource/ms_emoji

mt: espeak-ng-data/mt_dict
espeak-ng-data/mt_dict: dictsource/mt_list dictsource/mt_rules dictsource/mt_extra

my: espeak-ng-data/my_dict
espeak-ng-data/my_dict: dictsource/my_list dictsource/my_rules dictsource/my_extra dictsource/my_emoji

nci: espeak-ng-data/nci_dict
espeak-ng-data/nci_dict: dictsource/nci_list dictsource/nci_rules dictsource/nci_extra

ne: espeak-ng-data/ne_dict
espeak-ng-data/ne_dict: dictsource/ne_list dictsource/ne_rules dictsource/ne_extra dictsource/ne_emoji

nl: espeak-ng-data/nl_dict
espeak-ng-data/nl_dict: dictsource/nl_list dictsource/nl_rules dictsource/nl_extra dictsource/nl_emoji

no: espeak-ng-data/no_dict
espeak-ng-data/no_dict: dictsource/no_list dictsource/no_rules dictsource/no_extra

om: espeak-ng-data/om_dict
espeak-ng-data/om_dict: dictsource/om_list dictsource/om_rules dictsource/om_extra

or: espeak-ng-data/or_dict
espeak-ng-data/or_dict: dictsource/or_list dictsource/or_rules dictsource/or_extra dictsource/or_emoji

pa: espeak-ng-data/pa_dict
espeak-ng-data/pa_dict: dictsource/pa_list dictsource/pa_rules dictsource/pa_extra dictsource/pa_emoji

pap: espeak-ng-data/pap_dict
espeak-ng-data/pap_dict: dictsource/pap_list dictsource/pap_rules dictsource/pap_extra

pl: espeak-ng-data/pl_dict
espeak-ng-data/pl_dict: dictsource/pl_list dictsource/pl_rules dictsource/pl_extra dictsource/pl_emoji

pt: espeak-ng-data/pt_dict
espeak-ng-data/pt_dict: dictsource/pt_list dictsource/pt_rules dictsource/pt_extra dictsource/pt_emoji

py: espeak-ng-data/py_dict
espeak-ng-data/py_dict: dictsource/py_list dictsource/py_rules dictsource/py_extra

quc: espeak-ng-data/quc_dict
espeak-ng-data/quc_dict: dictsource/quc_list dictsource/quc_rules dictsource/quc_extra dictsource/quc_emoji

ro: espeak-ng-data/ro_dict
espeak-ng-data/ro_dict: dictsource/ro_list dictsource/ro_rules dictsource/ro_extra dictsource/ro_emoji

ru: espeak-ng-data/ru_dict
dictsource/ru_listx:
	ln -svf extra/ru_listx dictsource/
if HAVE_RU_EXTENDED_DICTIONARY
espeak-ng-data/ru_dict: dictsource/ru_list dictsource/ru_rules dictsource/ru_extra dictsource/ru_listx dictsource/ru_emoji
else
espeak-ng-data/ru_dict: dictsource/ru_list dictsource/ru_rules dictsource/ru_extra dictsource/ru_emoji
endif

sd: espeak-ng-data/sd_dict
espeak-ng-data/sd_dict: dictsource/sd_list dictsource/sd_rules dictsource/sd_extra dictsource/sd_emoji

shn: espeak-ng-data/shn_dict
espeak-ng-data/shn_dict: dictsource/shn_list dictsource/shn_rules dictsource/shn_extra

si: espeak-ng-data/si_dict
espeak-ng-data/si_dict: dictsource/si_list dictsource/si_rules dictsource/si_extra dictsource/si_emoji

sk: espeak-ng-data/sk_dict
espeak-ng-data/sk_dict: dictsource/sk_list dictsource/sk_rules dictsource/sk_extra dictsource/sk_emoji

sl: espeak-ng-data/sl_dict
espeak-ng-data/sl_dict: dictsource/sl_list dictsource/sl_rules dictsource/sl_extra dictsource/sl_emoji

sq: espeak-ng-data/sq_dict
espeak-ng-data/sq_dict: dictsource/sq_list dictsource/sq_rules dictsource/sq_extra dictsource/sq_emoji

dictsource/sr_emoji:
	@echo "  EMOJI     $@"
	@if test x"${CLDR_PATH}" = x ; then \
		touch $@ ; \
	else \
		tools/emoji dictsource/en_emoji sr_Latn ${CLDR_PATH} > $@ ; \
	fi

sr: espeak-ng-data/sr_dict
espeak-ng-data/sr_dict: dictsource/sr_list dictsource/sr_rules dictsource/sr_extra dictsource/sr_emoji

sv: espeak-ng-data/sv_dict
espeak-ng-data/sv_dict: dictsource/sv_list dictsource/sv_rules dictsource/sv_extra dictsource/sv_emoji

sw: espeak-ng-data/sw_dict
espeak-ng-data/sw_dict: dictsource/sw_list dictsource/sw_rules dictsource/sw_extra dictsource/sw_emoji

ta: espeak-ng-data/ta_dict
espeak-ng-data/ta_dict: dictsource/ta_list dictsource/ta_rules dictsource/ta_extra dictsource/ta_emoji

te: espeak-ng-data/te_dict
espeak-ng-data/te_dict: dictsource/te_list dictsource/te_rules dictsource/te_extra dictsource/te_emoji

tn: espeak-ng-data/tn_dict
espeak-ng-data/tn_dict: dictsource/tn_list dictsource/tn_rules dictsource/tn_extra

tr: espeak-ng-data/tr_dict
espeak-ng-data/tr_dict: dictsource/tr_list dictsource/tr_rules dictsource/tr_extra dictsource/tr_emoji

tt: espeak-ng-data/tt_dict
espeak-ng-data/tt_dict: dictsource/tt_list dictsource/tt_rules dictsource/tt_extra

ur: espeak-ng-data/ur_dict
espeak-ng-data/ur_dict: dictsource/ur_list dictsource/ur_rules dictsource/ur_extra dictsource/ur_emoji

vi: espeak-ng-data/vi_dict
espeak-ng-data/vi_dict: dictsource/vi_list dictsource/vi_rules dictsource/vi_extra dictsource/vi_emoji

zh: espeak-ng-data/zh_dict
dictsource/zh_listx:
	ln -svf extra/zh_listx dictsource/
if HAVE_ZH_EXTENDED_DICTIONARY
espeak-ng-data/zh_dict: dictsource/zh_list dictsource/zh_rules dictsource/zh_extra dictsource/zh_listx dictsource/zh_emoji
else
espeak-ng-data/zh_dict: dictsource/zh_list dictsource/zh_rules dictsource/zh_extra dictsource/zh_emoji
endif

zhy: espeak-ng-data/zhy_dict
dictsource/zhy_list:
	ln -svf extra/zhy_list dictsource/
if HAVE_ZHY_EXTENDED_DICTIONARY
espeak-ng-data/zhy_dict: dictsource/zhy_rules dictsource/zhy_extra dictsource/zhy_list dictsource/zhy_emoji
else
espeak-ng-data/zhy_dict: dictsource/zhy_rules dictsource/zhy_extra dictsource/zhy_emoji
endif
	cd dictsource && ESPEAK_DATA_PATH=$(PWD) LD_LIBRARY_PATH=../src:${LD_LIBRARY_PATH} ../src/espeak-ng --compile=yue && cd ..

dictsource/zhy_emoji:
	@echo "  EMOJI     $@"
	@if test x"${CLDR_PATH}" = x ; then \
		touch $@ ; \
	else \
		tools/emoji dictsource/en_emoji yue ${CLDR_PATH} > $@ ; \
	fi

##### mbrola:

mbrola: \
	espeak-ng-data/mbrola_ph/af1_phtrans \
	espeak-ng-data/mbrola_ph/ar1_phtrans \
	espeak-ng-data/mbrola_ph/ar2_phtrans \
	espeak-ng-data/mbrola_ph/ca_phtrans \
	espeak-ng-data/mbrola_ph/cr1_phtrans \
	espeak-ng-data/mbrola_ph/cs_phtrans \
	espeak-ng-data/mbrola_ph/de2_phtrans \
	espeak-ng-data/mbrola_ph/de4_phtrans \
	espeak-ng-data/mbrola_ph/de6_phtrans \
	espeak-ng-data/mbrola_ph/de8_phtrans \
	espeak-ng-data/mbrola_ph/ee1_phtrans \
	espeak-ng-data/mbrola_ph/en1_phtrans \
	espeak-ng-data/mbrola_ph/es_phtrans \
	espeak-ng-data/mbrola_ph/es3_phtrans \
	espeak-ng-data/mbrola_ph/es4_phtrans \
	espeak-ng-data/mbrola_ph/fr_phtrans \
	espeak-ng-data/mbrola_ph/gr1_phtrans \
	espeak-ng-data/mbrola_ph/gr2_phtrans \
	espeak-ng-data/mbrola_ph/grc-de6_phtrans \
	espeak-ng-data/mbrola_ph/hn1_phtrans \
	espeak-ng-data/mbrola_ph/hu1_phtrans \
	espeak-ng-data/mbrola_ph/ic1_phtrans \
	espeak-ng-data/mbrola_ph/id1_phtrans \
	espeak-ng-data/mbrola_ph/in_phtrans \
	espeak-ng-data/mbrola_ph/ir1_phtrans \
	espeak-ng-data/mbrola_ph/it1_phtrans \
	espeak-ng-data/mbrola_ph/it3_phtrans \
	espeak-ng-data/mbrola_ph/jp_phtrans \
	espeak-ng-data/mbrola_ph/la1_phtrans \
	espeak-ng-data/mbrola_ph/lt_phtrans \
	espeak-ng-data/mbrola_ph/ma1_phtrans \
	espeak-ng-data/mbrola_ph/mx1_phtrans \
	espeak-ng-data/mbrola_ph/mx2_phtrans \
	espeak-ng-data/mbrola_ph/nl_phtrans \
	espeak-ng-data/mbrola_ph/nz1_phtrans \
	espeak-ng-data/mbrola_ph/pl1_phtrans \
	espeak-ng-data/mbrola_ph/pt1_phtrans \
	espeak-ng-data/mbrola_ph/ptbr_phtrans \
	espeak-ng-data/mbrola_ph/ptbr4_phtrans \
	espeak-ng-data/mbrola_ph/ro1_phtrans \
	espeak-ng-data/mbrola_ph/sv_phtrans \
	espeak-ng-data/mbrola_ph/sv2_phtrans \
	espeak-ng-data/mbrola_ph/tl1_phtrans \
	espeak-ng-data/mbrola_ph/tr1_phtrans \
	espeak-ng-data/mbrola_ph/us_phtrans \
	espeak-ng-data/mbrola_ph/us3_phtrans \
	espeak-ng-data/mbrola_ph/vz_phtrans \
	espeak-ng-data/mbrola_ph/zh_phtrans

espeak-ng-data/mbrola_ph/af1_phtrans: phsource/mbrola/af1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/af1

espeak-ng-data/mbrola_ph/ar1_phtrans: phsource/mbrola/ar1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ar1

espeak-ng-data/mbrola_ph/ar2_phtrans: phsource/mbrola/ar2 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ar2

espeak-ng-data/mbrola_ph/ca_phtrans: phsource/mbrola/ca src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ca

espeak-ng-data/mbrola_ph/cr1_phtrans: phsource/mbrola/cr1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/cr1

espeak-ng-data/mbrola_ph/cs_phtrans: phsource/mbrola/cs src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/cs

espeak-ng-data/mbrola_ph/de2_phtrans: phsource/mbrola/de2 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/de2

espeak-ng-data/mbrola_ph/de4_phtrans: phsource/mbrola/de4 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/de4

espeak-ng-data/mbrola_ph/de6_phtrans: phsource/mbrola/de6 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/de6

espeak-ng-data/mbrola_ph/de8_phtrans: phsource/mbrola/de8 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/de8

espeak-ng-data/mbrola_ph/ee1_phtrans: phsource/mbrola/ee1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ee1

espeak-ng-data/mbrola_ph/en1_phtrans: phsource/mbrola/en1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/en1

espeak-ng-data/mbrola_ph/es_phtrans: phsource/mbrola/es src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/es

espeak-ng-data/mbrola_ph/es3_phtrans: phsource/mbrola/es3 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/es3

espeak-ng-data/mbrola_ph/es4_phtrans: phsource/mbrola/es4 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/es4

espeak-ng-data/mbrola_ph/fr_phtrans: phsource/mbrola/fr src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/fr

espeak-ng-data/mbrola_ph/gr1_phtrans: phsource/mbrola/gr1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/gr1

espeak-ng-data/mbrola_ph/gr2_phtrans: phsource/mbrola/gr2 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/gr2

espeak-ng-data/mbrola_ph/grc-de6_phtrans: phsource/mbrola/grc-de6 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/grc-de6

espeak-ng-data/mbrola_ph/hn1_phtrans: phsource/mbrola/hn1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/hn1

espeak-ng-data/mbrola_ph/hu1_phtrans: phsource/mbrola/hu1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/hu1

espeak-ng-data/mbrola_ph/ic1_phtrans: phsource/mbrola/ic1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ic1

espeak-ng-data/mbrola_ph/id1_phtrans: phsource/mbrola/id1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/id1

espeak-ng-data/mbrola_ph/in_phtrans: phsource/mbrola/in src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/in

espeak-ng-data/mbrola_ph/ir1_phtrans: phsource/mbrola/ir1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ir1

espeak-ng-data/mbrola_ph/it1_phtrans: phsource/mbrola/it1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/it1

espeak-ng-data/mbrola_ph/it3_phtrans: phsource/mbrola/it3 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/it3

espeak-ng-data/mbrola_ph/jp_phtrans: phsource/mbrola/jp src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/jp

espeak-ng-data/mbrola_ph/la1_phtrans: phsource/mbrola/la1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/la1

espeak-ng-data/mbrola_ph/lt_phtrans: phsource/mbrola/lt src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/lt

espeak-ng-data/mbrola_ph/ma1_phtrans: phsource/mbrola/ma1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ma1

espeak-ng-data/mbrola_ph/mx1_phtrans: phsource/mbrola/mx1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/mx1

espeak-ng-data/mbrola_ph/mx2_phtrans: phsource/mbrola/mx2 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/mx2

espeak-ng-data/mbrola_ph/nl_phtrans: phsource/mbrola/nl src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/nl

espeak-ng-data/mbrola_ph/nz1_phtrans: phsource/mbrola/nz1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/nz1

espeak-ng-data/mbrola_ph/pl1_phtrans: phsource/mbrola/pl1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/pl1

espeak-ng-data/mbrola_ph/pt1_phtrans: phsource/mbrola/pt1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/pt1

espeak-ng-data/mbrola_ph/ptbr_phtrans: phsource/mbrola/ptbr src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ptbr

espeak-ng-data/mbrola_ph/ptbr4_phtrans: phsource/mbrola/ptbr4 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ptbr4

espeak-ng-data/mbrola_ph/ro1_phtrans: phsource/mbrola/ro1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/ro1

espeak-ng-data/mbrola_ph/sv_phtrans: phsource/mbrola/sv src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/sv

espeak-ng-data/mbrola_ph/sv2_phtrans: phsource/mbrola/sv2 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/sv2

espeak-ng-data/mbrola_ph/tl1_phtrans: phsource/mbrola/tl1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/tl1

espeak-ng-data/mbrola_ph/tr1_phtrans: phsource/mbrola/tr1 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/tr1

espeak-ng-data/mbrola_ph/us_phtrans: phsource/mbrola/us src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/us

espeak-ng-data/mbrola_ph/us3_phtrans: phsource/mbrola/us3 src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/us3

espeak-ng-data/mbrola_ph/vz_phtrans: phsource/mbrola/vz src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/vz

espeak-ng-data/mbrola_ph/zh_phtrans: phsource/mbrola/zh src/espeak-ng
	mkdir -p espeak-ng-data/mbrola_ph
	ESPEAK_DATA_PATH=$(PWD) src/espeak-ng --compile-mbrola=phsource/mbrola/zh
