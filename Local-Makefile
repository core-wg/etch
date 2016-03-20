FILE=draft-vanderstok-core-fetch-patch-00
OPEN=$(word 1, $(wildcard /usr/bin/xdg-open /usr/bin/open /bin/echo))

all: txt html viewhtml

txt: $(FILE).txt

html: $(FILE).html

viewhtml: $(FILE).html
	$(OPEN) $<

pdf: $(FILE).pdf

%.xml: %.mkd
	kramdown-rfc2629 $< > $@

%.txt: %.xml
	xml2rfc $< --text

%.html: %.xml
	xml2rfc $< --html

%.ps: %.txt
	./fixff $<  | enscript --margins 76::76: -B -q -p $@

%.pdf: %.ps
	ps2pdf $< $@
