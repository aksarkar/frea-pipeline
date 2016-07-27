DATA := $(abspath ../..)
N := 2000
SEED := 0
PVE := 0.5
SHELL := bash
PLINK := plink --memory 4000
AUTOSOMES := 1 10 11 12 13 14 15 16 17 18 19 2 20 21 22 3 4 5 6 7 8 9

$(SEED).bed.gz: $(foreach c,$(AUTOSOMES),$(SEED).$(c).bed.gz)
	zcat $^ | bedtools sort | gzip >$@

$(SEED).%.assoc.linear: $(SEED).%.vcf.gz $(SEED).%.pheno $(SEED).%.eigenvec
	$(PLINK) --vcf $< --const-fid 0 --pheno $(word 2, $^) --covar $(word 3, $^) --allow-no-sex --linear --ci .95 --out $(SEED).$*

$(SEED).%.eigenval $(SEED).%.eigenvec: $(SEED).%.vcf.gz
	$(PLINK) --vcf $< --const-fid 0 --allow-no-sex --pca --out $(SEED).$*

$(SEED).%.vcf.gz $(SEED).%.pheno: $(SEED).pheno.pkl $(SEED).%.pkl
	frea-output-genotypes 0 $* $^ ALL.chr$*.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.{legend,haplotypes}.gz ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.sample <(zgrep -w chr$* decodeHotSpotFemale.txt.gz | gzip) $(SEED).$*

$(SEED).%.bed.gz: $(SEED).pheno.pkl $(SEED).%.pkl
	frea-simulate $(SEED) $^ ALL.chr$*.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.{legend,haplotypes}.gz ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.sample <(zgrep -w chr$* decodeHotSpotFemale.txt.gz | gzip) | sed s/,/chr$*/g | gzip >$@

$(SEED).pheno.pkl: $(foreach c,$(AUTOSOMES),$(SEED).$(c).pkl)
	frea-combine-pheno --pve $(PVE) $(SEED) $^ $@

$(SEED).%.pkl:
	frea-sample-events $(SEED) $(N) ALL.chr$*.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.{legend,haplotypes}.gz ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.sample <(zgrep -w chr$* decodeHotSpotFemale.txt.gz | gzip) $@

pre:
	parallel -X ln -sf {} . ::: $(DATA)/1kg/*.{gz,sample} $(DATA)/decode/*.txt.gz

joblist:
	parallel --dry-run make SEED={} ::: $$(seq 0 99) >$@

clean:
	find \( -name "*.pkl" -o -name "*.*.bed.gz" \) | parallel -X rm

.DELETE_ON_ERROR:
