DATA := $(abspath ../../data)
RESULTS := $(abspath ..)
CUTOFFS := $(RESULTS)/rrplot/core-features/all-traits/cutoffs
SUMMARY := $(RESULTS)/impg/out
MHC := $(DATA)/mask/mhc+5mb.bed.gz
CUTOFFS := $(RESULTS)/rrplot/core-features/all-traits/cutoffs

INTERSECT := bedtools intersect -sorted -nobuf
EXPAND = python -m frea.ld.expand $< | bedtools sort | gzip

PROP := maf tss-dist ld
FEATURES := core-features
SHELL := bash

export LC_ALL=C

define FDR
import frea.algorithms
with open('$<') as f:
	data = [line.split() for line in f]
	for record in frea.algorithms.benjamini_hochberg(data, key=lambda x: float(x[7]), fdr=0.017):
		print(*record)
endef
export FDR

%-significant.in: %.in
	echo "$$FDR"
	python -c "$$FDR" >$@

%-table.txt: $(FEATURES)/%.in %-permutation-stats.txt.gz
	join <(awk '{print $$1"|"$$2"|"$$3, $$0}' $< | sort) <(zcat $(word 2, $^) | awk '{print $$1"|"$$2"|"$$3, $$0}' | sort) | awk '{sub(".*-", "", $$2); print toupper($$2), $$3, $$4, $$5, $$6, $$16, $$7, $$8, $$9, $$10}' | sort -k1,1 -k2,2 -k3,3 >$@

%-permutation-stats.txt.gz:
	awk '{gsub(".bed.gz", "", $$0); a = "EnhClusters"} $$3 == "" {$$3 = 0} $$2 ~ /^E/ {a = "Enh"} {print $$1, a, $$2, $$3, $$4}' tabulate.o* | sort -k1,1 -k2,2 -k3,3 | gzip >$@

joblist: $(CUTOFFS)
	parallel --dry-run -C" " '$(INTERSECT) -c -a {1} -b {3} | awk '"'"'BEGIN {n = 0; c = 0} $$5 > {2} {n += 1; if ($$6 > 0) {c += 1}} END {print "{1/}", "{3/}", c, n - c}'"'" :::: $< ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@

$(FEATURES)/joblist: $(CUTOFFS) table.txt.gz
	mkdir -p $(FEATURES)
	parallel --dry-run -C" " frea-matched :::: $(word 1,$^) ::: $(abspath $(word 2,$^)) ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@

bed: $(notdir $(wildcard $(SUMMARY)/*.bed.gz))

%.bed.gz: $(SUMMARY)/%.bed.gz
	$(INTERSECT) -v -a $< -b $(MHC) | gzip >$@

table.txt.gz: $(PROP:=.txt.gz)
	frea-build-table $^ | gzip >$@

maf.txt.gz:
	parallel "zcat $(DATA)/1kg/ALL.chr{}.*.legend.gz | python -c 'from frea.matched import *; extract_maf({})'" ::: $$(seq 1 22) | sort -k1 | gzip >$@

tss-dist.txt.gz: $(DATA)/1kg/1kg-eur.bed.gz
	bedtools intersect -sorted -nobuf -v -a $< -b $(MHC) | bedtools closest -d -a stdin -b $(DATA)/encode/Gencodev10_CAGE_TSS_clusters_May2012.gff.gz | awk -vOFS='\t' '{print $$4, int($$NF/1000)}' | sort -k1 | gzip >$@

ld.txt.gz:
	zcat $(DATA)/ld/*.gz | awk '{count[$$1]++; count[$$2]++} END {for (c in count) {print c, count[c]}}' | sort -k1 | gzip >$@

.DELETE_ON_ERROR:
