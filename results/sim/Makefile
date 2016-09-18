DATA := $(abspath ../..)
SEED := 0
PVE := 0.18
SNPTEST = snptest -chunk 1000 -cov_all_continuous -frequentist 1 -method expected -o $@

AUTOSOMES := 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22
C := 58C NBS BD CAD CD HT RA T1D T2D
COHORT := {58C,NBS,BD,CAD,CD,HT,RA,T1D,T2D}

%.gz: %
	gzip $<

$(SEED).bed.gz: $(foreach c,$(AUTOSOMES),$(SEED).$(c).txt.gz)
	zcat $^ | gzip >$@

$(SEED).%.txt: $(SEED).sentinel
	frea-output-summary -w 1000 $* $(COHORT){-$(SEED).pheno,_$*.gen.gz} >$@

$(SEED).sentinel: $(SEED).pheno.pkl
	frea-output-pheno $< $(SEED) $(COHORT)-clean.sample
	touch $@

$(SEED).pheno.pkl: $(foreach c,$(AUTOSOMES),$(SEED).$(c).pkl)
	frea-combine-pheno --pve=$(PVE) $^ $@

$(SEED).%.pkl:
	frea-sample-gvs --p-causal=0.74 -- $(SEED) $(COHORT){-clean.sample,_$*.gen.gz} $@

joblist.1:
	parallel -j1 --dry-run make SEED={1} AUTOSOMES={2} ::: $$(seq 1 10) ::: $(AUTOSOMES) >$@

joblist:
	parallel -j1 --dry-run make SEED={1} AUTOSOMES={2} {1}.{2}.pkl ::: $$(seq 1 10) ::: $(AUTOSOMES) >$@

pre:
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/*.gen.gz $(DATA)/wtccc/vcf/*-clean.sample

clean:
	find \( -name "*.pkl" -o -name "*.*.bed.gz" \) | parallel -X rm

.DELETE_ON_ERROR:
