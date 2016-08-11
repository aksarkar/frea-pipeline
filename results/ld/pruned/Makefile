RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS)/../data)
GWAS := $(wildcard $(RESULTS)/impg/out/*.bed.gz)

PARALLEL := parallel -j4 --halt soon,fail=1

THRESH := 0
CHR := 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22

$(notdir $(GWAS)): $(foreach c, $(CHR), chr$(c)-pruned.gz)
	python -m frea.ld.collect | gzip >$@

%-pruned.gz: shard
	python -m frea.ld.prune $*.txt.gz $(DATA)/ld/$*.txt.gz $(THRESH) | gzip >$@

shard: $(GWAS)
	python -m frea.ld.shard $<

fdr-0.05.ranks: num-loci.txt
	$(PARALLEL) -C" " 'zgrep -v inf {1} | sort -k4 | bedtools groupby -g 4 -c 5 -o max | sort -k2gr | awk '"'"'$$2 <= -log(0.05 * NR / {2}) / log(10) {print "{1}", NR; exit}'"'" :::: $< >$@

num-loci.txt:
	$(PARALLEL) 'echo {} $$(zgrep -v inf {} | sort -k4 | bedtools groupby -g 4 -c 5 -o max | wc -l)' ::: *.bed.gz

.PHONY: shard

.DELETE_ON_ERROR:
