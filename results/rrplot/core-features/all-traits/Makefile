RESULTS := $(abspath ../../..)
DATA := $(abspath $(RESULTS:results=data))

INTERSECT := bedtools intersect -sorted -nobuf

cutoffs: Enh.ranks
	parallel -C" " -j1 '$(INTERSECT) -wa -a $(RESULTS)/impg/out/{1}.bed.gz -b $(RESULTS)/restrict/all-studies-impg.bed.gz | $(INTERSECT) -v -a stdin -b $(DATA)/mask/mhc+5mb.bed.gz | sort -k5gr | awk '"'"'NR == {3} {print "{1}", $5}'"'" :::: $< >$@

Enh.txt.gz:
	cat rrplot.o* | gzip >$@

joblist:
	parallel --dry-run frea-rrplot -c -i $(RESULTS)/restrict/all-studies-impg.bed.gz -s $(DATA)/mask/mhc+5mb.bed.gz {1} {2} ::: $(RESULTS)/impg/out/*.bed.gz ::: $(DATA)/roadmap/core-features/Enh/*.gz >joblist

