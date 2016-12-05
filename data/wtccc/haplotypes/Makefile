DATA := $(abspath ../..)
AFFY := $(DATA)/arrays/Affy-NSP-STY-b37.58-v4.strand.gz
1KG := $(DATA)/1kg
REF := integrated_phase1_v3.20101123.snps_indels_svs.genotypes
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
SRC := $(wildcard *_chiamo.gz)

GCTA = gcta64-1.24 --thread-num $(NSLOTS) --out $*
PLINK = plink --memory 4096 --out $(basename $@)
EXCLUDE = awk '/^[^\#]/ {print $$3, $$3}' $< >$@

all: $(SRC:_chiamo.gz=.haps.gz) $(CASES:=.eigenvec)

%.eigenvec: %.grm.bin
	$(GCTA) --grm $* --pca 10 >/dev/null

all.grm.bin: all.grm.txt
	$(GCTA) all --mgrm $< --make-grm >/dev/null

all.grm.txt:
	parallel -j1 echo chr{} ::: $$(seq -w 1 22) >$@

%.grm.bin: %-merge.bed
	$(GCTA) --bfile $(basename $<) --make-grm >/dev/null

chr%-merge.bed: chr%.merge
	$(PLINK) --merge-list $< --make-bed >/dev/null

chr%.merge:
	parallel -j1 echo {}_$* ::: 58C NBS BD CAD CD HT RA T1D T2D >$@

pheno: $(CASES:=-merge.pheno)

%.pheno: %.bed
	awk '{print $$1, $$2, $$6}' $(<:bed=fam) >$@

%-merge.bed: %.merge %_01.bed
	$(PLINK) --merge-list $*.merge --make-pheno $*_01.fam '*' --make-bed >/dev/null

%.merge:
	parallel -j1 echo {1}_{2} ":::" 58C NBS $* ::: $$(seq -w 1 22) >$@

%.haps.gz: %.haps
	gzip $<

%.haps: C = $(shell echo $* | sed s/^.*_0\\?//)
%.haps: %.snp.strand.exclude %.bed
	shapeit -B $* --exclude-snp $< -R $(1KG)/ALL.chr$(C).$(REF).nosing.{haplotypes,legend}.gz $(1KG)/ALL.$(REF).sample -T 8 -O $* -L $*.phase >/dev/null

%.snp.strand.exclude: COHORT = $(word 1, $(subst _, ,$*))
%.snp.strand.exclude: CHROMOSOME = $(word 2, $(subst _, ,$*))
%.snp.strand.exclude: C = $(shell echo $* | sed s/^.*_0\\?//)
%.snp.strand.exclude: %.bed
	shapeit -check -B $* -R $(1KG)/ALL.chr$(C).$(REF).nosing.{haplotypes,legend}.gz $(1KG)/ALL.$(REF).sample -L $* >/dev/null || true

%.bed: COHORT = $(word 1, $(subst _, ,$*))
%.bed: C = $(shell echo $* | sed s/^.*_0\\?//)
%.bed: %.orient.gz exclude sample
	$(PLINK) --gen $< --sample $(COHORT).sample --hard-call-threshold .01 --remove $(COHORT).exclude --exclude snps.exclude --chr $(C) --geno 0.5 --make-bed >/dev/null

%.orient.gz: %.lift.gz strand
	gtool -O --g $< --strand $(word 2, $(subst _, ,$*)).strand --og $(@:.gz=) --log $(@:.orient.gz=.gtool.log) >/dev/null

define LIFT
awk -vc=$(CHROMOSOME) 'BEGIN {c = int(c)} FNR == NR && $$2 == c {snps[$$1] = $$3} FNR < NR {if ($$1 in snps) {$$3 = snps[$$1]; $$2 = $$1; $$1 = c; print}}' <(zcat $^) <(zcat $(word 2, $^)) | sort -k3n | gzip >$@
endef
export LIFT

%.lift.gz: CHROMOSOME = $(word 2, $(subst _, ,$*))
%.lift.gz: $(AFFY) %_chiamo.gz
	bash -c "$$LIFT"

sample: $(COHORT:=.sample)
	touch sample

define CLEAN
awk 'BEGIN {
    print "ID_1", "ID_2", "missing", "sex"
    print 0, 0, 0, "D"
}

($$5 == "12656D9" && $$1 == "WTCCC69833") ||
($$5 == "12656G8" && $$1 == "WTCCC69712") ||
($$5 == "12705A1" && $$1 == "WTCCC59114") ||
($$5 == "12709H2" && $$1 == "WTCCC61926") ||
($$5 == "12699H1" && $$1 == "WTCCC66712") {
    $$1 = $$1 "_2"
}

NR > 2 {
    print $$1, $$1, 0, $$2
}' $< >$@
endef
export CLEAN

SAMPLE = bash -c "$$CLEAN"

58C.sample: Affx_20060707fs1_sample_58C.txt
	$(SAMPLE)

NBS.sample: Affx_20061220fs1_sample_NBS.txt
	$(SAMPLE)

%.sample: Affx_20070205fs1_sample_%.txt
	$(SAMPLE)

exclude: $(COHORT:=.exclude) snps.exclude
	touch exclude

58C.exclude: exclusion-list-05-02-2007.txt
	$(EXCLUDE)

NBS.exclude: exclusion-list.txt
	$(EXCLUDE)

%.exclude: exclusion-list-05-02-2007-%.txt
	$(EXCLUDE)

snps.exclude: exclusion-list-snps-26_04_2007.txt
	awk '/^[1-9]/ {print $$2}' $< >$@

strand:
	zcat $(AFFY) | awk '{print $$3, $$5 >sprintf("%02d.strand", $$2)}'
	touch strand

pre: manifest
	parallel -X ln -s {} . :::: $< || true

joblist.3: sample exclude strand
	parallel -j1 --dry-run make DATA=$(DATA) {1}_{2}.lift.gz ::: $(COHORT) ::: $$(seq -w 1 22) >$@

joblist.2: sample exclude strand
	parallel -j1 --dry-run make chr{}.grm.bin ::: $$(seq -w 1 22) >$@

joblist.1: sample exclude strand
	parallel -j1 --dry-run make {1}_{2}.haps.gz ::: $(COHORT) ::: $$(seq -w 1 22) >$@

joblist: sample exclude strand
	parallel -j1 --dry-run make DATA=$(DATA) {1}_{2}.bed ::: $(COHORT) ::: $$(seq -w 1 22) >$@

clean:
	find . \( -type f -o -type l \) \! \( -name "Makefile" -o -name "manifest" -o -name "*.haps.gz" -o -name "*.eigenvec" -o -name "*.sample" -o -name "*.exclude" \) | parallel -X rm

.PHONY: pre clean pheno

.DELETE_ON_ERROR:
