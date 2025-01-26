.PHONY: all clean structure pdfs

COLORS=Rosewater Flamingo Pink Mauve Red Maroon Peach Yellow Green Teal Sky Sapphire Blue Lavender Overlay2 Overlay1 Overlay0
FLAVORS=Latte Frappe Mocha Macchiato
THEMES=$(basename $(subst /usr/local/texlive/2024/texmf-dist/tex/latex/beamer/beamertheme,,$(wildcard /usr/local/texlive/2024/texmf-dist/tex/latex/beamer/beamertheme[A-Z]*.sty))) default

COMBINATIONS=$(foreach t,$(THEMES),$(foreach c,$(COLORS),$(foreach f,$(FLAVORS),$(t)-$(c)-$(f))))

PDFS=$(addsuffix .pdf,$(COMBINATIONS))
TEXS=$(addsuffix .tex,$(COMBINATIONS))


IMAGES=$(patsubst %,merged/%.jpg,$(COMBINATIONS)) \
$(foreach c, $(COMBINATIONS), $(foreach size, large small, $(foreach page, 01 02 12 17 18, $(size)/$(c)-$(page).png)))

all: structure $(IMAGES) index.html ../assets/preview.webp

../assets/preview.webp: $(addprefix ../assets/,$(addsuffix .webp, $(FLAVORS)))
	catwalk -o "$@" $(addprefix ../assets/,$(addsuffix .webp, $(FLAVORS)))

$(addprefix ../assets/,$(addsuffix .webp, $(FLAVORS))): ../assets/%.webp: large/CambridgeUS-Rosewater-%-01.ppm
	convert "$<" "$@"

index.html: structure build_web.sh $(IMAGES)
	bash build_web.sh

pdfs: $(PDFS)

texs: $(TEXS)

structure: large small merged

large small merged:
	mkdir $@

large/%-01.ppm large/%-02.ppm: %.pdf
	pdftoppm -f  1 -l  2 "$<" large/$(basename $<)

large/%-12.ppm: %.pdf
	pdftoppm -f 12 -l 12 "$<" large/$(basename $<)

large/%-17.ppm large/%-18.ppm: %.pdf
	pdftoppm -f 17 -l 18 "$<" large/$(basename $<)


merged/%.jpg: large/%-01.ppm large/%-02.ppm large/%-12.ppm large/%-17.ppm large/%-18.ppm
	pnmcat -lr $^ | pnmscale 0.142857143 | pnmtojpeg --quality=85 --optimize >  "$@"

small/%.png: large/%.ppm
	pnmscale 0.428571429 < "$<" | pnmtopng -compression 3 > "$@"

large/%.png: large/%.ppm
	pnmtopng -compression 3 < "$<" > "$@"

%.tex: template.tex.tmpl
	@head -n 23 "$<" > "$@"
	@echo "$@" | sed 's/\([^-]*\)-\([^-]*\)-\([^-\.]*\)\..*/\\usetheme{\1}\n\\usecolortheme[style=\3,accent=\2]{catppuccin}\n/' >> "$@"
	@tail -n 716 "$<" >> "$@"
	@touch -r template.tex.tmpl "$@"

%.pdf: %.tex 
	latexmk -quiet -pdf "$<"
	rm $(addprefix $(basename $<)., aux log nav out snm toc fdb_latexmk fls)

clean:
	-$(RM) *.aux *.log *.nav *.out *.snm *.toc *.fls *.fdb_latexmk

distclean: clean
	-$(RM) -r large small merged individual *.pdf *.tex *.html
