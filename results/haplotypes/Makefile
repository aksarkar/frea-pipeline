CODE := $(HOME)/code/impute2/pre
AFFY := /broad/compbio/aksarkar/data/arrays/Affy-NSP-STY-b37.58-v4.strand.gz
1KG := /broad/compbio/aksarkar/data/1kg
REF := integrated_phase1_v3.20101123.snps_indels_svs.genotypes
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
SRC := $(wildcard *_chiamo.gz)

GCTA = gcta64-1.24 --thread-num 8 --out $*
PLINK = plink --memory 4096 --out $(basename $@)
PRINT = parallel -C" " -j1 /bin/echo -e

all: $(SRC:_chiamo.gz=.haps.gz) $(CASES:=.eigenvec)

%.hsq: %.params $(addprefix %.grm.,bin N id) $(addprefix %.eigen,val vec) %.pheno
	java -jar ~/.local/src/PCGCRegression/java-pcgc/PCGCRegression.jar $< >$@

%.permuted.params: %.permuted.pheno
	$(PRINT) $* >$*.grms; $(PRINT) T1D >$*.subtractvecs; $(PRINT) "grmlist=$*.grms" "subtractvecs=$*.subtractvecs" "phenos=$*.permuted.pheno" "covars=$*.eigenvec" "prevalence=0.005" >$@

params: prevalences.txt
	$(PRINT) {1} ">" {1}.grms :::: $<
	$(PRINT) {1} ">" {1}.subtractvecs :::: $<
	$(PRINT) 'grmlist={1}.grms\\nsubtractvecs={1}.subtractvecs\\nphenos={1}.pheno\\ncovars={1}.eigenvec\\nprevalence={2}' ">" {1}.params :::: $<

%.pheno: %-clean.fam
	awk '{print $$1, $$2, $$6 - 1}' $< >$@

$(addprefix %.eigen,val vec): $(addprefix %.grm.,bin N id)
	$(GCTA) --grm $* --grm-cutoff .05 --pca 10 >/dev/null

$(addprefix %.grm.,bin N id): $(addprefix %-temp.grm.,bin N id)
	$(GCTA) --grm $*-temp --grm-cutoff .05 --make-grm >/dev/null

$(addprefix %-temp.grm.,bin N id): $(addprefix %-clean.,bed bim fam)
	$(GCTA)-temp --bfile $(basename $<) --make-grm >/dev/null

$(addprefix %-clean.,bed bim fam): %-merge.exclude $(addprefix %-merge.,bed bim fam)
	$(PLINK) --bfile $*-merge --exclude $< --mind .01 --geno .01 --maf .01 --hwe .05 --make-bed >/dev/null

.PRECIOUS: %-clean.bed %-clean.bim %-clean.fam %.eigenvec %.eigenval %.pheno

%-merge.exclude: %-merge.missing
	awk '$$5 < .05 {print $$2}' $< >$@

%-merge.missing: $(addprefix %-merge.,bed bim fam)
	$(PLINK) --bfile $*-merge --test-missing >/dev/null

$(addprefix %-merge.,bed bim fam): %.merge %_01.bed mhc.txt
	$(PLINK) --bfile $*_01 --merge-list $*.merge --exclude range mhc.txt --make-pheno $*_01.fam '*' --make-bed >/dev/null

mhc.txt:
	echo "6 23000000 39400000 mhc" >$@

%.merge:
	parallel -j1 echo {1}_{2} ":::" 58C NBS $* ::: $$(seq -w 1 22) | grep -v $*_01 >$@

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
	$(PLINK) --gen $< --sample $(COHORT).sample --hard-call-threshold .01 --remove $(COHORT).exclude --exclude snps.exclude --chr $(C) --make-bed >/dev/null

%.orient.gz: %.lift.gz strand
	gtool -O --g $< --strand $(word 2, $(subst _, ,$*)).strand --og $(@:.gz=) --log $(@:.orient.gz=.gtool.log) >/dev/null

%.lift.gz: CHROMOSOME = $(word 2, $(subst _, ,$*))
%.lift.gz: %_chiamo.gz positions
	zcat $< | python $(CODE)/lift.py $(CHROMOSOME) $(CHROMOSOME).positions | sort -k3n | gzip >$@

sample: $(COHORT:=.sample)
	touch sample

58C.sample: Affx_20060707fs1_sample_58C.txt
	awk -f $(CODE)/sample.awk $< >$@

NBS.sample: Affx_20061220fs1_sample_NBS.txt
	awk -f $(CODE)/sample.awk $< >$@

%.sample: Affx_20070205fs1_sample_%.txt
	awk -f $(CODE)/sample.awk $< >$@

exclude: $(COHORT:=.exclude) snps.exclude
	touch exclude

58C.exclude: exclusion-list-05-02-2007.txt
	awk '/^[^#]/ {print $$1, $$3}' $< >$@

NBS.exclude: exclusion-list.txt
	awk '/^[^#]/ {print $$1, $$3}' $< >$@

%.exclude: exclusion-list-05-02-2007-%.txt
	awk '/^[^#]/ {print $$1, $$3}' $< >$@

snps.exclude: exclusion-list-snps-26_04_2007.txt
	awk '/^[1-9]/ {print $$2}' $< >$@

positions:
	zcat $(AFFY) | awk '{print $$1, $$3 >sprintf("%02d.positions", $$2)}'
	touch positions

strand:
	zcat $(AFFY) | awk '{print $$3, $$5 >sprintf("%02d.strand", $$2)}'
	touch strand

pre: manifest
	parallel -X ln -sf {} . :::: $<

joblist:
	parallel -j1 --dry-run make {1}_{2}.bed ::: $(COHORT) ::: $$(seq -w 1 22) >$@

.DELETE_ON_ERROR:
