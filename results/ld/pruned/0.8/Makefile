RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS)/../data)
GWAS := $(wildcard $(RESULTS)/impg/out/*.bed.gz)

THRESH := 0
CHR := 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 

$(notdir $(GWAS)): $(foreach c, $(CHR), chr$(c)-pruned.gz)
	python -m frea.ld.collect | gzip >$@

%-pruned.gz: shard
	python -m frea.ld.prune $*.txt.gz $(DATA)/ld/$*.txt.gz $(THRESH) | gzip >$@

shard: $(GWAS)
	python -m frea.ld.shard $<

.PHONY: shard

.DELETE_ON_ERROR:
