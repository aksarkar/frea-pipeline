DATA := $(abspath ../../data)
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
AUTOSOMES := 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22

joblist: $(CASES:=.joblist)
	cat $^ >$@

%.joblist: %.sample
	parallel --dry-run "$(MAKE) DATA=$(DATA) $*_{}.txt.gz" ::: $(AUTOSOMES) >$@

%.gz: %
	gzip $<

%.txt: COHORT = $(word 1, $(subst _, ,$*))
%.txt: CHROMOSOME = $(word 2, $(subst _, ,$*))
%.txt: %.gen.gz
	snptest -data {$(COHORT),58C,NBS}{_$(CHROMOSOME).gen.gz,.sample} -pheno pheno -cov_all_continuous -frequentist 1 -method ml -o $@ >$*.log

define PREPARE
FNR == NR {
	h = "ID_1 ID_2 missing pheno PC1"
	h2 = "0 0 0 B C"
	k = $$1 "|" $$2;
	v = $$3;
	for (i = 4; i <= NF; i++) {
		v = v FS $$i
		h = h FS "PC" (i - 2)
		h2 = h2 FS "C"
	}
	eigen[k] = v
	if (FNR == 1) {
		print h
		print h2
	}
}
FNR < NR {
	k = $$1 "|" $$2
	if (k in eigen) {
		print $$1, $$2, 0, pheno, eigen[k]
	}
}
endef
export PREPARE

%.sample: $(addprefix $(DATA)/wtccc/haplotypes/%,.eigenvec .sample)
	awk -v pheno=1 "$$PREPARE" $^ >$@
	awk -v pheno=0 "$$PREPARE" $< $(DATA)/wtccc/haplotypes/58C.sample >58C.sample
	awk -v pheno=0 "$$PREPARE" $< $(DATA)/wtccc/haplotypes/NBS.sample >NBS.sample

.PRECIOUS: %.sample

define FILTER
import csv, itertools, sys
window_size = 5000000
pos = (int(line.split()[2]) for line in sys.stdin)
for k, g in itertools.groupby(pos, key=lambda x: x // window_size):
    if len(list(g)) > 100:
        print('{} {}'.format(k * window_size, (k + 1) * window_size - 1))
endef
export FILTER

%.windows: $(DATA)/wtccc/haplotypes/%.haps.gz
	zcat $< | python -c "$$FILTER" >$@

pre:
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/*.gen.gz

.PHONY: clean

.DELETE_ON_ERROR:
