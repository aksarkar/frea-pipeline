DATA := /broad/compbio/aksarkar/data
RESULTS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results
SUMMARY := $(RESULTS)/impg/out
MHC := $(DATA)/mask/mhc+5mb.bed.gz

FEATURES := core-features

define FDR
import frea.algorithms
with open('$*.txt') as f:
	data = [line.split() for line in f]
	for record in frea.algorithms.benjamini_hochberg(data, key=lambda x: float(x[7])):
		print(*record)
endef
export FDR

%-significant.in:
	python -c "$$FDR" >$@

$(FEATURES)/joblist: $(RESULTS)/rrplot/core-features/all-traits/cutoffs $(RESULTS)/struct/table.txt.gz
	mkdir -p $(FEATURES)
	parallel --dry-run -C" " frea-matched :::: $(word 1,$^) ::: $(word 2,$^) ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@

bed: $(notdir $(wildcard $(SUMMARY)/*.bed.gz))

%.bed.gz: $(SUMMARY)/%.bed.gz
	bedtools intersect -sorted -nobuf -v -a $< -b $(MHC) | gzip >$@
