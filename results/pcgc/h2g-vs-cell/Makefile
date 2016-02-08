BASE := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results
DATA := /broad/compbio/aksarkar/data
CODE := $(HOME)/code
SRCDIR := $(DATA)/roadmap/core-features/Enh
PHENO := T1D
SRC := $(wildcard $(SRCDIR)/*.bed.gz)

GWAS := $(BASE)/motifs/by-inst/top.bed.gz

THRESH := $(shell awk 'NR == 1 {print $$2}' $(BASE)/matched/core-features/cutoffs)

PARALLEL := parallel -j1
INTERSECT := bedtools intersect -sorted -nobuf

MAKE_GRM = if [ -s $*.tags ]; then gcta64-1.24 --bfile $(PHENO) --keep $(PHENO).grm.id --extract $< --make-grm --out $(@:.grm.bin=) &>$(@:.grm.bin=).log; else touch $@; fi
PRINT := $(PARALLEL) echo ":::"

export LC_ALL=C

%.pdf: %.txt
	Rscript $(CODE)/wtccc/gcta/plot_hsq_vs_annotations.R $<

%.snps: $(GWAS) $(SRCDIR)/%.bed.gz
	$(INTERSECT) -a $(word 1, $^) -b $(word 2, $^) | cut -f4 >$@

%.tags: $(PHENO)-clean.snps %.snps top.ld.gz
	if [ -s $*.snps ]; then python $(CODE)/ld/1kg/tag.py $^ >$@; else touch $@; fi

$(addprefix %.grm.,bin N.bin id): %.tags $(PHENO).grm.id
	$(MAKE_GRM)

$(addprefix %.c.grm.,bin N.bin id): %.tags $(PHENO).grm.id
	$(MAKE_GRM:extract=exclude)

%.params:
	$(PRINT) $* $*.c >$*.grms; $(PRINT) $(PHENO) $(PHENO) >$*.subtractvecs; $(PRINT) "grmlist=$*.grms" "subtractvecs=$*.subtractvecs" "phenos=$(PHENO).pheno" "covars=$(PHENO).eigenvec" "prevalence=0.005" >$@

%.hsq: %.params %.grm.bin %.c.grm.bin
	if [ -s $*.grm.bin ]; then java -jar $(HOME)/.local/src/PCGCRegression/java-pcgc/PCGCRegression.jar $< &>$@; else echo 'G0 0 0 0' >$@; fi

h2g-vs-$(notdir $(SRCDIR)).txt: $(notdir $(SRC:.bed.gz=.hsq))
	$(PARALLEL) awk -f $(CODE)/wtccc/gcta/h2g-vs-annotation.awk ":::" $^ >$@

joblist:
	parallel --dry-run make SRCDIR=$(SRCDIR) SRC={} ::: $(SRC) >$@

top.ld.gz: $(PHENO)-clean.snps
	python $(CODE)/ld/1kg/filter.py $< $(GWAS) /broad/compbio/pouyak/popgen/proj/1kg/phase1/corr/EUR.txt.gz | gzip >$@

$(PHENO)-clean.snps: $(PHENO).bed
	cut -f2 $(<:.bed=.bim) >$@

$(PHENO).bed: $(PHENO)-clean.names
	plink --bfile $(PHENO)-clean --update-name $< --make-bed --out $(@:.bed=)

$(PHENO)-clean.names: pre
	python $(CODE)/util/join_bed_1kg.py <$(@:.names=.bim) >$@

pre:
	ln -sf /broad/hptmp/aksarkar/haplotypes/$(PHENO){,-clean}.* . && touch $@

clean:
	parallel -X rm -f ::: $(notdir $(SRC:bed.gz=*)) pre

.PHONY: clean

.DELETE_ON_ERROR:
