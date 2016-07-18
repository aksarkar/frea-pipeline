DATA := $(abspath ../..)
SEED := 0

%.gz: %
	gzip $<

$(SEED).bed.gz: $(foreach c,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22,$(SEED).$(c).txt.gz)
	parallel --halt now,fail=1 -j1 python -c "\""'from frea.summary.process import *; oxstats_snptest_to_ucsc_bed('"'"$(SEED).{}.txt.gz"'", "'"{}"'"')'"\"" ::: $$(seq -w 1 22) | gzip >$@

.INTERMEDIATE: $(foreach c,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22,$(SEED).$(c).txt.gz)

define PREPARE
FNR == NR {
	h = "PC1"
	h2 = "C"
	k = $$1 "|" $$2;
	v = $$3;
	for (i = 4; i <= NF; i++) {
		v = v FS $$i
		h = h FS "PC" (i - 2)
		h2 = h2 FS "C"
	}
	eigen[k] = v
}
FNR < NR && FNR == 1 {
	print $$0, h
}
FNR < NR && FNR == 2 {
	print $$0, h2
}
FNR < NR && FNR >= 2 && $$1 "|" $$2 in eigen {
	print $$0, eigen[$$1 "|" $$2]
}
endef
export PREPARE

$(SEED).%.txt: $(SEED).covars
	snptest -chunk 10000 -data 58C_$*.gen.gz $< -pheno pheno -cov_all_continuous -frequentist 1 -method expected -o $(SEED).$*.txt >$(SEED).$*.log

$(SEED).covars: T1D.eigenvec $(SEED).pheno
	awk "$$PREPARE" $^ >$@

$(SEED).pheno: 58C-clean.sample
	zcat 58C_*.gen.gz | python -m frea.simulation $(SEED) - 58C-clean.sample >$@

.INTERMEDIATE: $(SEED).pheno

%-clean.sample: %.exclude %.sample
	awk 'FNR == NR {drop[$$2] = 1} $$2 in drop {next} FNR < NR {print}' $^ >$@

pre:
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/58C*.gen.gz $(DATA)/wtccc/haplotypes/*.{eigenvec,exclude,sample}

joblist.1:
	parallel --dry-run "make SEED={1.} {1.}.{2}.txt.gz" ::: *.covars ::: $$(seq -w 1 22) >$@

joblist:
	parallel --dry-run make SEED={} ::: $$(seq 0 99) >$@

clean:
	find \( -name "snptest.*" -o -name "*.txt.gz" -o -name "*.log" \) | parallel -X rm

.DELETE_ON_ERROR:
