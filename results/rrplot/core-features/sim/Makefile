RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS:results=data))

INTERSECT := bedtools intersect -sorted -nobuf

%.cutoffs: %.ranks
	parallel -C" " -j1 '$(INTERSECT) -v -a $(RESULTS)/ld/pruned/{1}.bed.gz -b $(DATA)/mask/mhc+5mb.bed.gz | sort -k4 | bedtools groupby -g 4 -c 5 -o max | sort -k2gr | awk '"'"'NR == {3} {print "{1}.bed.gz", $$2}'"'" :::: $< >$@

joblist:
	parallel --dry-run frea-rrplot -c -l weight -s $(DATA)/mask/mhc+5mb.bed.gz {1} {2} 100 ::: $(RESULTS)/sim/*.bed.gz ::: $(DATA)/roadmap/core-features/Enh/*.bed.gz >$@
