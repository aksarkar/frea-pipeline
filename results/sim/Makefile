DATA := $(abspath ../..)
SEED := 0

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

$(SEED).covars: T1D.eigenvec $(SEED).pheno
	awk "$$PREPARE" $^ >$@

$(SEED).pheno: 58C-clean.sample
	zcat $(DATA)/wtccc/1kg-phase1/58C_*.gen.gz | python -m frea.simulation $(SEED) - 58C-clean.sample >$@

.INTERMEDIATE: $(SEED).pheno

%-clean.sample: %.exclude %.sample
	awk 'FNR == NR {drop[$$2] = 1} $$2 in drop {next} FNR < NR {print}' $^ >$@

pre:
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/58C*.gen.gz $(DATA)/wtccc/haplotypes/*.{eigenvec,exclude,sample}

joblist.1:
	parallel --dry-run "snptest -chunk 10000 -data $(DATA)/wtccc/1kg-phase1/58C_{1}.gen.gz {2} -pheno pheno -cov_all_continuous -frequentist 1 -method em -o {2.}.{1}.txt >{2.}.{1}.log" ::: $$(seq 1 22) ::: *.covars >$@

joblist:
	parallel --dry-run make DATA=$(DATA) SEED={} ::: $$(seq 0 99) >$@

.DELETE_ON_ERROR:
