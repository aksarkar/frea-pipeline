DATA := $(abspath ../..)
N := 50000
DOWNSAMPLE := 100
SEED := 0
PVE := 0.5
SHELL := bash
AUTOSOMES := 1 10 11 12 13 14 15 16 17 18 19 2 20 21 22 3 4 5 6 7 8 9

PREFIX := $(SEED)-$(N)-$(PVE)

PLINK = plink --memory 4000 --vcf $< --const-fid 0 --allow-no-sex --out $(SEED).$*
REF = ALL.chr$*.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.{legend,haplotypes}.gz ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.sample $*.hotspots.gz
SIM = frea-simulate --covars $(word 3, $^) --chunk 100 chr$* $(word 1, $^) $(word 2, $^)

$(PREFIX).bed.gz: $(foreach c,$(AUTOSOMES),$(PREFIX).$(c).bed.gz)
	zcat $^ | bedtools sort | gzip >$@

$(PREFIX).%.assoc.linear: $(PREFIX).%.vcf.gz $(PREFIX).%.pheno $(PREFIX).%.eigenvec
	$(PLINK) --pheno $(word 2, $^) --covar $(word 3, $^) --linear --ci .95

$(PREFIX).%.eigenval $(PREFIX).%.eigenvec: $(PREFIX).%.vcf.gz
	$(PLINK) --pca

.INTERMEDIATE: $(foreach c, $(AUTOSOMES), $(PREFIX).$(c).eigenvec)

$(PREFIX).%.vcf.gz $(PREFIX).%.pheno: $(PREFIX).pheno.pkl $(PREFIX).%.pkl
	frea-output-genotypes $* $^ $(PREFIX).$*

.PRECIOUS: $(foreach c, $(AUTOSOMES), $(PREFIX).$(c).vcf.gz)

ldsc: $(PREFIX).ldsc

.PHONY: ldsc

$(PREFIX).ldsc: $(PREFIX).txt.gz
	ldsc.py --h2 $< --ref-ld-chr ./ --w-ld-chr ./ >$@

$(PREFIX).txt.gz: $(foreach c, $(AUTOSOMES), $(PREFIX).$(c).txt.gz)
	zcat $^ | gzip >$@

$(PREFIX).%.txt.gz: $(PREFIX).pheno.pkl $(PREFIX).%.pkl $(PREFIX).svd.pkl
	$(SIM) --output-sumstats | gzip >$@

$(PREFIX).%.bed.gz: $(PREFIX).pheno.pkl $(PREFIX).%.pkl $(PREFIX).svd.pkl
	$(SIM) | gzip >$@

$(PREFIX).svd.pkl: $(foreach c,$(AUTOSOMES),$(PREFIX).$(c).pkl)
	frea-pca --n-per-block=$(DOWNSAMPLE) $^ $@

$(PREFIX).pheno.pkl: $(foreach c,$(AUTOSOMES),$(PREFIX).$(c).pkl)
	frea-combine-pheno --pve=$(PVE) $^ $@

$(PREFIX).%.pkl: hotspots
	frea-sample-events $(SEED) $(N) $(REF) $@

hotspots: $(foreach c,$(AUTOSOMES),$(c).hotspots.gz)
	touch $@

%.hotspots.gz:
	zgrep -w chr$* decodeHotSpotFemale.txt.gz | gzip >$@

.PRECIOUS: $(foreach c,$(AUTOSOMES),$(c).hotspots.gz)

pre:
	parallel -X ln -sf {} . ::: $(DATA)/1kg/*.{gz,sample} $(DATA)/decode/*.txt.gz $(DATA)/ldsc/*.{ldscore.gz,m_5_50}

clean:
	find \( -name "*.pkl" -o -name "*.*.bed.gz" \) | parallel -X rm

.DELETE_ON_ERROR:
