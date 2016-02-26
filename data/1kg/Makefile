CURL := curl -sfS
1KG_PREFIX := ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing
CHROMOSOME := $(foreach c,$(shell seq 1 22),$(c).bed.gz)

1kg-eur.bed.gz: $(CHROMOSOME)
	zcat $^ | bedtools sort | gzip >$@

%.bed.gz: $(subst ALL,ALL.chr%,$(1KG_PREFIX)).legend.gz stage
	zcat $< | awk -v chromosome=$* -f $(HOME)/code/struct/snp/names.awk | gzip >$@

stage:
	$(CURL) 'https://mathgen.stats.ox.ac.uk/impute/$(1KG_PREFIX).tgz' | tar xf -
	touch stage

.INTERMEDIATE: $(CHROMOSOME)

.DELETE_ON_ERROR:
