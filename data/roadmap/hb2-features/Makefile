P := 10
SRCDIR := /broad/compbio/aksarkar/data/roadmap/honeybadger2/DNase/p$(P)/enh/BED_files_per_cluster
OUTDIR := EnhClusters-p$(P)
SRC := $(wildcard $(SRCDIR)/*.bed)

$(OUTDIR)/%.bed.gz: $(SRCDIR)/cluster_%.bed
	bedtools sort -i $< | gzip >$@

all: $(foreach feature,$(SRC),$(OUTDIR)/$(patsubst cluster_%.bed,%.bed.gz,$(notdir $(feature))))
	echo $(foreach feature,$(SRC),$(patsubst cluster_%,%.gz,$(feature)))
