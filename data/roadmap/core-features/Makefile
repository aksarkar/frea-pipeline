all:
	parallel -C' ' 'zgrep -E "Enh\\b" {} | cut -f1-3 | bedtools sort | gzip >Enh/$(basename {} _15_coreMarks_mnemonics.bed.gz).bed.gz' ::: $</*_mnemonics.bed.gz
