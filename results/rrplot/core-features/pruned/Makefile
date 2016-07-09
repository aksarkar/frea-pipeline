RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS:results=data))
KEEP := -i $(RESULTS)/restrict/all-studies-impg.bed.gz

INTERSECT := bedtools intersect -sorted -nobuf

%.cutoffs: %.ranks
	parallel -C" " -j1 '$(INTERSECT) -v -a $(RESULTS)/ld/pruned/{1}.bed.gz -b $(DATA)/mask/mhc+5mb.bed.gz | sort -k4 | bedtools groupby -g 4 -c 5 -o max | sort -k2gr | awk '"'"'NR == {3} {print "{1}", $$2}'"'" :::: $< >$@

joblist:
	parallel --dry-run frea-rrplot -c -l weight $(KEEP) -s $(DATA)/mask/mhc+5mb.bed.gz {1} {2} 100 ::: $(RESULTS)/ld/pruned/$*/*.bed.gz ::: $(DATA)/roadmap/core-features/Enh/*.bed.gz >$@
