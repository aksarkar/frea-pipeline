Enh: ../core
	mkdir -p $@
	parallel -C' ' 'zgrep -E "Enh\\b" {} | cut -f1-3 | bedtools sort | gzip >$@/$$(basename {} _15_coreMarks_mnemonics.bed.gz).bed.gz' ::: $</*_mnemonics.bed.gz
