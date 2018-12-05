MAIN := ietf-103-tsvarea-transport-proxies
OUTDIR := build

.DEFAULT_GOAL := $(OUTDIR)/$(MAIN).pdf

$(OUTDIR)/$(MAIN).pdf: $(wildcard *.tex) | $(OUTDIR)
	pdflatex --output-directory=$(OUTDIR) $(MAIN).tex

clean: ; $(RM) $(wildcard *.log) $(wildcard $(OUTDIR)/*)

$(OUTDIR): ; mkdir -p $@
