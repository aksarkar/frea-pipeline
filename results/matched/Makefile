DATA := $(abspath ../../data)
RESULTS := $(abspath ..)
SUMMARY := $(RESULTS)/impg/out
MHC := $(DATA)/mask/mhc+5mb.bed.gz
LD := /broad/compbio/pouyak/popgen/proj/1kg/phase1/corr/EUR+.txt.gz

INTERSECT := bedtools intersect -sorted -nobuf
EXPAND = python -m frea.ld.expand $< | bedtools sort | gzip

PROP := maf tss-dist ld
FEATURES := core-features

export LC_ALL=C

define FDR
import frea.algorithms
with open('$<') as f:
	data = [line.split() for line in f]
	for record in frea.algorithms.benjamini_hochberg(data, key=lambda x: float(x[7])):
		print(*record)
endef
export FDR

%-significant.in: %.in
	echo "$$FDR"
	python -c "$$FDR" >$@

$(FEATURES)/joblist: $(RESULTS)/rrplot/core-features/all-traits/cutoffs table.txt.gz
	mkdir -p $(FEATURES)
	parallel --dry-run -C" " frea-matched :::: $(word 1,$^) ::: $(abspath $(word 2,$^)) ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@

bed: $(notdir $(wildcard $(SUMMARY)/*.bed.gz))

control/joblist: $(RESULTS)/rrplot/core-features/all-traits/cutoffs table.txt.gz $(addprefix control/,$(notdir $(wildcard $(SUMMARY)/*.bed.gz)))
	parallel --dry-run -C" " frea-matched :::: $(word 1,$^) ::: $(abspath $(word 2,$^)) ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@

control/%.bed.gz: %.bed.gz coding.bed.gz %-loci.bed.gz
	$(INTERSECT) -v -a $< -b $(word 2, $^) | $(INTERSECT) -v -a stdin -b $(word 3, $^) | gzip >$@

coding.bed.gz: exonic.bed.gz $(DATA)/mask/tss+2kb.bed.gz
	python -m frea.multimerge $^ | cut -f1-3 | bedtools merge | bedtools sort | gzip >$@

%-loci.bed.gz: $(DATA)/1kg/EUR+.txt.gz %.bed.gz
	zcat $(word 2, $^) | awk -vFS='\t' -vOFS='\t' '{if ($$5 > 8 - log(5)) {print $$0, 1} else {print $$0, 0}}' | $(EXPAND) >$@

exonic.bed.gz: $(DATA)/1kg/EUR+.txt.gz
	zcat $(DATA)/mask/gencode.v10.annotation.gtf.gz | awk '$$3 == "exon" && /protein_coding/ && /gene_status "KNOWN"/ && /transcript_status "KNOWN"/' | $(INTERSECT:-sorted=-c) -a ../restrict/all-studies-impg.bed.gz -b stdin | $(EXPAND) >$@

%.bed.gz: $(SUMMARY)/%.bed.gz
	$(INTERSECT) -v -a $< -b $(MHC) | gzip >$@

table.txt.gz: $(PROP:=.txt.gz)
	frea-build-table $^ | gzip >$@

maf.txt.gz:
	parallel "zcat $(DATA)/1kg/ALL.chr{}.*.legend.gz | python -c 'from frea.matched import *; extract_maf({})'" ::: $$(seq 1 22) | sort -k1 | gzip >$@

tss-dist.txt.gz: $(DATA)/1kg/1kg-eur.bed.gz
	bedtools intersect -sorted -nobuf -v -a $< -b $(MHC) | bedtools closest -d -a stdin -b $(DATA)/encode/Gencodev10_CAGE_TSS_clusters_May2012.gff.gz | awk -vOFS='\t' '{print $$4, int($$NF/1000)}' | sort -k1 | gzip >$@

ld.txt.gz:
	zcat $(LD) | awk '{count[$$1]++; count[$$2]++} END {for (c in count) {print c, count[c]}}' | sort -k1 | gzip >$@

.DELETE_ON_ERROR:
