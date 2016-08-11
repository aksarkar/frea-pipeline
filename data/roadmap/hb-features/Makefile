SRCDIR := ../honeybadger
SRC := $(wildcard $(SRCDIR)/*.bed.gz)

all: $(patsubst cluster_%.bed.gz,%.bed.gz,$(notdir $(SRC)))
	mkdir -p EnhClusters
	mv *.bed.gz EnhClusters

%.bed.gz: $(SRCDIR)/cluster_%.bed.gz
	bedtools sort -i $< | bedtools merge -i stdin | gzip >$@
