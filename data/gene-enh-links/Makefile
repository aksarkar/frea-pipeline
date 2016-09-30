CURL := curl -sfSL

links.fdr.01.bed.gz: Union_link_sig_FDR01_Sep.txt.gz
	zcat $< | awk -vOFS='\t' '$$2 ~ /^chr/ {print $$2, $$3, $$4, $$6}' | bedtools sort | gzip >$@

Union_link_sig_FDR01_Sep.txt.gz:
	$(CURL) -O "https://www.broadinstitute.org/~anshul/projects/roadmap/integrative/enh_gene_links/Union_link_sig_FDR01_Sep.txt.gz"

