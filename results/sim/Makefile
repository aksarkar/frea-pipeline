DATA := $(abspath ../..)
SEED := 0
PVE := 0.18
HOLDOUT := 

AUTOSOMES := 01 10 11 12 13 14 15 16 17 18 19 02 20 21 22 03 04 05 06 07 08 09
C := 58C NBS BD CAD CD HT RA T1D T2D
COHORT := {58C,NBS,BD,CAD,CD,HT,RA,T1D,T2D}

$(SEED).bed.gz: $(foreach c,$(AUTOSOMES),$(SEED).$(c).txt.gz)
	parallel -j1 "zcat {} | awk 1" ::: $^ | LC_ALL=C sort -k1,1 -k2,2n -k3,3n | gzip >$@

failed:
	parallel -j1 'zcat {} | awk -vFS="\t" '"'"'NF != 5 {print "{}"; exit 1}'"'" ::: *.txt.gz >$@

.PHONY: failed

$(SEED).%.txt.gz: $(SEED).sentinel
	frea-output-summary -w 1000 -s $(SEED) $(HOLDOUT) $* $(COHORT){-$(SEED).pheno,_$*.gen.gz} | gzip >$@

$(SEED).sentinel: $(SEED).pheno.pkl
	frea-output-pheno $< $(SEED) $(COHORT)-clean.sample
	touch $@

$(SEED).pheno.pkl: $(foreach c,$(AUTOSOMES),$(SEED).$(c).pkl)
	frea-combine-pheno --pve=$(PVE) $^ $@

$(SEED).%.pkl:
	frea-sample-gvs --p-causal=0.74 -- $(SEED) $(COHORT){-clean.sample,_$*.gen.gz} $@ 2>$(@:.pkl=.log)

joblist.1:
	parallel -j1 --dry-run make SEED={1} AUTOSOMES={2} HOLDOUT="'$(HOLDOUT)'" {1}.{2}.txt.gz ::: $$(seq 1 10) ::: $(AUTOSOMES) >$@

joblist:
	parallel -j1 --dry-run make SEED={1} AUTOSOMES={2} {1}.{2}.pkl ::: $$(seq 1 10) ::: $(AUTOSOMES) >$@

bed:
	parallel make SEED={} ::: $$(seq 1 10)

.PHONY: bed

pre:
	parallel -X ln -sf {} . ::: $(DATA)/wtccc/1kg-phase1/*.gen.gz $(DATA)/wtccc/vcf/*-clean.sample
	touch $@

clean:
	find \( -name "*.pkl" -o -name "*.*.bed.gz" -o -name "*.pheno" -o -name "*.txt.gz" \) | parallel -X rm

.DELETE_ON_ERROR:
