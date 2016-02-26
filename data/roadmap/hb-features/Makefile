SRCDIR := ../honeybadger
SRC := $(wildcard $(SRCDIR)/*.bed)

all: $(patsubst cluster_%.bed,%.bed.gz,$(notdir $(SRC)))

%.bed.gz: $(SRCDIR)/cluster_%.bed
	cut -f1-3 $< | bedtools sort | bedtools merge -i stdin | gzip >$@
