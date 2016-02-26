P := 10
SRCDIR := ../honeybadger2-impute-p$(P)
OUTDIR := EnhClusters-p$(P)
SRC := $(wildcard $(SRCDIR)/*.bed)

$(OUTDIR)/%.bed.gz: $(SRCDIR)/cluster_%.bed
	bedtools sort -i $< | gzip >$@

all: $(foreach feature,$(SRC),$(OUTDIR)/$(patsubst cluster_%.bed,%.bed.gz,$(notdir $(feature))))
