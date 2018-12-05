MAIN := main
OUTDIR := build

.DEFAULT_GOAL := $(OUTDIR)/$(MAIN).pdf

$(OUTDIR)/$(MAIN).pdf: $(OUTDIR) $(wildcard *.tex)
	pdflatex --output-directory=$(OUTDIR) $(MAIN).tex

clean: ; $(RM) $(wildcard *.log) $(wildcard $(OUTDIR)/*)

$(OUTDIR): ; mkdir -p $@
