CODE := $(HOME)/code/impute2/pre
AFFY := /broad/compbio/aksarkar/data/arrays/Affy-NSP-STY-b37.58-v4.strand.gz
1KG := /broad/compbio/aksarkar/data/1kg
REF := integrated_phase1_v3.20101123.snps_indels_svs.genotypes
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
SRC := $(wildcard *_chiamo.gz)
SHELL := bash

GCTA = gcta64-1.24 --thread-num 8 --out $*
PLINK = plink --memory 4000 --out $(basename $@)
PRINT = parallel -j1 echo :::

all: eigen haps qc

eigen: $(CASES:=.eigenvec)

haps: $(SRC:_chiamo.gz=.haps.gz)

qc: $(CASES:=-clean.bed)

%.hsq: %.params
	java -jar ~/.local/src/PCGCRegression/java-pcgc/PCGCRegression.jar $< >$@

%.permuted.params: %.grm.bin %.eigenvec %.permuted.pheno
	$(PRINT) $* >$*.grms; $(PRINT) T1D >$*.subtractvecs; $(PRINT) "grmlist=$*.grms" "subtractvecs=$*.subtractvecs" "phenos=$*.permuted.pheno" "covars=$*.eigenvec" "prevalence=0.005" >$@

%.params: %.grm.bin %.eigenvec %.pheno 
	$(PRINT) $* >$*.grms; $(PRINT) T1D >$*.subtractvecs; $(PRINT) "grmlist=$*.grms" "subtractvecs=$*.subtractvecs" "phenos=$*.pheno" "covars=$*.eigenvec" "prevalence=0.005" >$@

%.permuted.pheno: %.pheno
	python $(HOME)/code/wtccc/gcta/permute.py 0 <$< >$@

%.pheno: %-clean.fam
	awk '{print $$1, $$2, $$6 - 1}' $< >$@

%.eigenvec: %.grm.bin
	$(GCTA) --grm $* --grm-cutoff .05 --pca 10 >/dev/null

%.grm.bin: %-temp.grm.bin
	$(GCTA) --grm $*-temp --grm-cutoff .05 --make-grm >/dev/null

%-temp.grm.bin: %-clean.bed
	parallel -j1 $(GCTA)-{} --bfile $(basename $<) --chr {} --make-grm >/dev/null ::: $$(seq 1 22)
	parallel -j1 echo $*-{} ::: $$(seq 1 22) >%-grms.txt
	$(GCTA)-temp --mgrm %-grms.txt --make-grm >/dev/null

%-clean.bed %-clean.bim %-clean.fam: %-merge.exclude %-merge.bed
	$(PLINK) --bfile $*-merge --exclude $< --geno .05 --maf .01 --hwe .05 --make-bed >/dev/null

.PRECIOUS: %-clean.bed

%-merge.exclude: %-merge.missing
	awk '$$5 < .05 {print $$2}' $< >$@

%-merge.missing: %-merge.bed
	$(PLINK) --bfile $*-merge --test-missing >/dev/null

%-merge.bed: %.merge %_01.bed mhc.txt
	$(PLINK) --bfile $*_01 --merge-list $*.merge --exclude range mhc.txt --make-pheno $*_01.fam '*' --make-bed >/dev/null

mhc.txt:
	echo "6 23000000 39400000 mhc" >$@

%.merge:
	parallel -j1 echo {1}_{2} ":::" 58C NBS $* ::: $$(seq -w 1 22) | grep -v $*_01 >$@

joblist:
	parallel -j1 --dry-run make {1}_{2}.bed ::: $(COHORT) ::: $$(seq -w 1 22) >$@

%.bed: COHORT = $(word 1, $(subst _, ,$*))
%.bed: C = $$(echo $* | sed s/^.*_0\\?//)
%.bed: %.gen.gz %.keep
	$(PLINK) --gen <(zcat $< | awk -v chromosome=$(C) -f $(CODE:pre=post)/names.awk) --sample $*.sample --hard-call-threshold .01 --oxford-single-chr $(C) --extract $*.keep --make-bed >/dev/null

%.keep: C = $$(echo $* | sed s/^.*_0\\?//)
%.keep: %.snptest
	awk -v chromosome=chr$(C) -v field=position -v info=0.6 -f $(HOME)/code/wtccc/snptest/post.awk $< | awk '{print $$4}' >$@

%.snptest: COHORT = $(word 1, $(subst _, ,$*))
%.snptest: C = $$(echo $* | sed s/^.*_//)
%.snptest: %.gen.gz
	snptest -summary_stats_only -missing_code -9 -data $< $*.sample NBS_$(C).gen.gz NBS_$(C).sample 58C_$(C).gen.gz 58C_$(C).sample -o $@ -log $*.log >/dev/null

sample:
	ln -sf $(foreach c,$(COHORT),/broad/hptmp/aksarkar/haplotypes/$(c).sample) .

.DELETE_ON_ERROR:
