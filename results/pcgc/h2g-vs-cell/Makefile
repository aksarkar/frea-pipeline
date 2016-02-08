SHELL := bash
GWAS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/haplotypes
SRCDIR := /broad/compbio/aksarkar/data/roadmap/hb-features/EnhClusters
SUMMARY := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/out
CUTOFFS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/rrplot/core-features/all-traits/cutoffs
MHC := /broad/compbio/aksarkar/data/mask/mhc+5mb.bed.gz
LD := /broad/compbio/pouyak/popgen/proj/1kg/phase1/corr/EUR+.txt.gz

CODE := $(HOME)/code
SRC := $(wildcard $(SRCDIR)/*.bed.gz)
CASES := BD CAD CD RA T1D T2D
MAPPING := "BD pgc-bip" "CAD cardiogram-cad" "CD iibdgc-cd" "RA stahl-ra" "T1D bradfield-t1d" "T2D diagram-t2d"
INTERSECT := bedtools intersect -sorted -nobuf
GCTA := gcta64-1.24 --thread-num 8
MAKE_GRM = if [ -s $*.tags ]; then $(GCTA) --bfile $(PHENO) --keep $(GWAS)/$(PHENO).grm.id --extract $< --make-grm --out $(@:.grm.bin=) &>$(@:.grm.bin=).log; else touch $@; fi
PRINT := parallel -j1 echo ":::"

export LC_ALL=C

%.pdf: %.txt
	Rscript $(CODE)/wtccc/gcta/plot_hsq_vs_annotations.R $<

%.hsq: %.grms %.grm.bin %.c.grm.bin
	if [ -s $*.grm.bin ]; then $(GCTA) --reml --mgrm $*.grms --pheno $(GWAS)/$(PHENO).pheno --qcovar $(GWAS)/$(PHENO).eigenvec --prevalence $(PREVALENCE) --out $*; else $(PRINT) "V(G1)/Vp_L 0 0" "Pval 1" >$@; fi

$(addprefix %.grm.,bin N.bin id): %.tags $(GWAS)/$(PHENO).grm.id
	$(MAKE_GRM)

$(addprefix %.c.grm.,bin N.bin id): %.tags $(GWAS)/$(PHENO).grm.id
	$(MAKE_GRM:extract=exclude)

%.grms: %.grm.bin %.c.grm.bin
	$(PRINT) $* $*.c >$@

%.tags: $(PHENO)-clean.snps %.snps $(PHENO).ld.gz
	if [ -s $*.snps ]; then python -m frea.ld.tag $^ >$@; else touch $@; fi

%-clean.snps: %.bed
	cut -f2 $(<:.bed=.bim) >$@

$(addprefix %.,bed bim fam): %-clean.names
	plink --bfile $(GWAS)/$*-clean --update-name $< --make-bed --out $*

%-clean.names: $(GWAS)/%-clean.bim
	python -c "from frea.summary.process import *; fix_plink_bim_names()" <$< >$@

h2g-vs-$(notdir $(SRCDIR)).txt:
	python -m frea.heritability.postprocess >$@

hsq: $(addprefix $(PHENO).,$(notdir $(SRC:.bed.gz=.hsq)))

joblist.2:
	parallel -C" " --dry-run 'make PHENO={1} PREVALENCE={2} SRC={3} {1}.$$(basename {3} .bed.gz).hsq' :::: $(GWAS)/prevalences.txt ::: $(SRCDIR)/*.bed.gz | grep -v HT >$@

joblist.1:
	parallel -C" " --dry-run '$(INTERSECT) -a {2}.bed.gz -b {3} | cut -f4 >{1}.$$(basename {3} .bed.gz).snps' ::: $(MAPPING) ::: $(SRCDIR)/*.bed.gz >$@

joblist: $(CASES:=-clean.snps)
	parallel --dry-run -C" " "python -m frea.ld.filter {1}-clean.snps {2}.bed.gz $(LD) | gzip >{1}.ld.gz" ::: $(MAPPING) >$@

names: $(CASES:=-clean.snps)

top: $(CUTOFFS)
	parallel -C" " -j1 '$(INTERSECT) -v -a $(SUMMARY)/{1} -b $(MHC) | awk '"'"'$$5 > {2}'"'"' | gzip >{1}' :::: $<

clean:
	parallel -X rm -f ::: $(foreach c,$(CASES),$(addprefix $(c).*.,hsq tags c.* grm* log))

.PRECIOUS: $(addprefix %.,bed bim fam) %.hsq %.tags

.PHONY: clean

.DELETE_ON_ERROR:
