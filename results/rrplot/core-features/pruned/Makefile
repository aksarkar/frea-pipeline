RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS:results=data))

all: $(foreach r,1 8,0.$(r)/joblist)

%/joblist:
	mkdir -p $*
	parallel --dry-run frea-rrplot -c -l weight -i $(RESULTS)/restrict/all-studies-impg.bed.gz -s $(DATA)/mask/mhc+5mb.bed.gz {1} {2} 100 ::: $(RESULTS)/ld/pruned/$*/*.bed.gz ::: $(DATA)/roadmap/core-features/Enh/*.bed.gz >$@
