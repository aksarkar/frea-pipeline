DATA := $(abspath ../..)
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
SRC := $(wildcard *_chiamo.gz)
SHELL := bash

GCTA = gcta64-1.24 --thread-num 8 --out $*
PLINK = plink --memory 4000 --out $(basename $@)

all: $(CASES=.vcf.bgz)

%.vcf.gz: %-merge.exclude %-merge.bed
	$(PLINK:.vcf=) --bfile $*-merge --exclude $< --recode vcf bgz >/dev/null

%-clean.bed %-clean.bim %-clean.fam: %-merge.exclude %-merge.bed
	$(PLINK) --bfile $*-merge --exclude $< --geno .05 --maf .01 --hwe .05 --make-bed >/dev/null

.PRECIOUS: %-clean.bed

%-merge.exclude: %-merge.missing
	awk '$$5 < .05 {print $$2}' $< >$@

58C-merge.missing:
	touch $@

%-merge.missing: %-merge.bed
	$(PLINK) --bfile $*-merge --test-missing >/dev/null

58C-merge.bed: 58C.merge
	$(PLINK) --merge-list $< --make-bed

%-merge.bed %-merge.fam %-merge.bim: %.merge %_01.bed
	$(PLINK) --merge-list $*.merge --make-pheno $*_01.fam '*' --make-bed

58C.merge:
	parallel echo 58C_{} ::: $$(seq -w 1 22) >$@

%.merge:
	parallel -j1 echo {1}_{2} ":::" 58C NBS $* ::: $$(seq -w 1 22) >$@

joblist:
	parallel -j1 --dry-run make {1}_{2}.bed ::: $(COHORT) ::: $$(seq -w 1 22) >$@

define NAMES
{
    rsid = $$2
    position = $$3
    a0 = $$4
    a1 = $$5
    if (length(a0) <= length(a1)) {
        start = position
        end = position
        if (length(a0) == length(a1)) {
            delta = a1
        }
        else {  # insertion
            delta = substr(a1, length(a0))
        }
    }
    else {  # deletion
        start = position + length(a1)
        end = position + length(a0) - length(a1)
        delta = ""
    }

    $$2 = rsid "|chr" chromosome "|" start "|" end "|" delta
    print
}
endef
export NAMES

## Hack: This doesn't seem to actually be local to the recipe?
58C_%.bed: C = $$(echo $* | sed s/^0//)
58C_%.bed: 58C_%.gen.gz
	$(PLINK) --gen <(zcat $< | awk -v chromosome=$(C) "$$NAMES") --sample 58C-clean.sample --hard-call-threshold .01 --oxford-single-chr $(C) --make-bed >/dev/null

NBS_%.bed: C = $$(echo $* | sed s/^0//)
NBS_%.bed: NBS_%.gen.gz
	$(PLINK) --gen <(zcat $< | awk -v chromosome=$(C) "$$NAMES") --sample NBS-clean.sample --hard-call-threshold .01 --oxford-single-chr $(C) --make-bed >/dev/null

%.bed: COHORT = $(word 1, $(subst _, ,$*))
%.bed: CHR = $$(echo $* | sed s/^.*_0\\?//)
%.bed: %.gen.gz %.keep
	$(PLINK) --gen <(zcat $< | awk -v chromosome=$(CHR) "$$NAMES") --sample $(COHORT)-clean.sample --hard-call-threshold .01 --oxford-single-chr $(CHR) --extract $*.keep --make-bed >/dev/null

%.keep: COHORT = $(word 1, $(subst _, ,$*))
%.keep: C = $$(echo $* | sed s/^.*_//)
%.keep: %.gen.gz sample
	python -m frea.info {$(COHORT),NBS,58C}_$(C).gen.gz >$@

sample: $(COHORT:=-clean.sample)
	touch $@

%-clean.sample: %.exclude %.sample
	awk 'FNR == NR {drop[$$2] = 1} $$2 in drop {next} FNR < NR {print}' $^ >$@

pre: 
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/*.gen.gz $(DATA)/wtccc/haplotypes/*.{exclude,sample}

.DELETE_ON_ERROR:
