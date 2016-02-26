CURL := curl -sfS

mhc+5mb.bed.gz:
	echo -e "chr6\t23000000\t39400000" | gzip >$@

gencode.v10.annotation.gtf.gz:
	$(CURL) --ftp-pasv -O "ftp://ftp.sanger.ac.uk/pub/gencode/Gencode_human/release_10/gencode.v10.annotation.gtf.gz"

tss+%.bed.gz: human.hg19.genome
	$(CURL) --ftp-pasv 'ftp://genome.crg.es/pub/Encode/data_analysis/TSS/Gencodev10_CAGE_TSS_clusters_May2012.gff.gz' | bedtools slop -l $* -r 0 -g $< | cut -f1,4,5 | bedtools merge | gzip >$@

human.hg19.genome:
	$(CURL) -O 'https://raw.githubusercontent.com/arq5x/bedtools2/6cca09e007858ebf81d668995785ea95f589f8fa/genomes/human.hg19.genome'

