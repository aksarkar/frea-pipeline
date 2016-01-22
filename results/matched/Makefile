CODE := $(HOME)/code/enr/generic/matched
DATA := /broad/compbio/aksarkar/data
BASE := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results
GWAS := $(BASE)/impg/out/*.bed.gz

FEATURES := core-features

$(FEATURES)/joblist: $(BASE)/rrplot/core-features/all-traits/cutoffs $(BASE)/struct/table.txt.gz
	mkdir -p $(FEATURES)
	parallel --dry-run -C" " bash $(CODE)/test.sh :::: $(word 1,$^) ::: $(word 2,$^) ::: $(DATA)/roadmap/$(FEATURES)/*/*.bed.gz >$@
