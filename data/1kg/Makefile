CURL := curl -sfS
TAR: tar --strip-components 1 xf
1KG_PREFIX := ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing
CHROMOSOME := $(foreach c,$(shell seq 1 22),$(c).bed.gz)

1kg-eur.bed.gz: $(CHROMOSOME)
	zcat $^ | bedtools sort | gzip >$@

define NAMES
{
    rsid = $$1
    position = $$2
    a0 = $$3
    a1 = $$4
    if (length(a0) <= length(a1)) {
        start = position
        end = position
        if (length(a0) == length(a1)) {
            delta = a1
        }
        else {  # insertion
            delta = substr(a1, length(a0))
        }
    }
    else {  # deletion
        start = position + length(a1)
        end = position + length(a0) - length(a1)
        delta = ""
    }

    id = rsid "|chr" chromosome "|" start "|" end "|" delta
    print "chr"chromosome, position - 1, position + length(a0) - 1, id, -1
}
endef
export NAMES

%.bed.gz: $(subst ALL,ALL.chr%,$(1KG_PREFIX)).legend.gz stage
	zcat $< | awk -v chromosome=$* '$$NAMES' | gzip >$@

stage:
	$(CURL) 'https://mathgen.stats.ox.ac.uk/impute/$(1KG_PREFIX).tgz' | $(TAR) -
	touch stage

.INTERMEDIATE: $(CHROMOSOME)

.DELETE_ON_ERROR:
