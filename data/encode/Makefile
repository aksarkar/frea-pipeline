CURL := curl -sfS

all: clusters.txt $(addsuffix .txt.gz,enrichments expr-corr motifs-toscan) $(addsuffix .bed.gz,matches motif-background)

clusters.txt:
	awk '{split($$2, a, "[(),]*"); for (i in a) {if (a[i] != "") {print a[i], $$1}}}' /broad/compbio/pouyak/motifs/verts/data/motifs/tfh/table/motifs-clust.txt >$@

enrichments.txt.gz:
	$(CURL) -O "http://egg2.wustl.edu/roadmap/data/byDataType/motifanalysis/pouyak/clustered/enrichments.txt.gz"

expr-corr.txt.gz:
	$(CURL) -O "http://egg2.wustl.edu/roadmap/data/byDataType/motifanalysis/pouyak/clustered/expr-corr.txt.gz"

motifs-toscan.txt.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/motifs-toscan.txt.gz" | zcat >$@

matches.bed.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/matches.txt.gz" | zgrep -v "_C[0-9]" | awk -vOFS="\t" '{sub(/_8mer/, "", $1); print $2, $3-1, $4, $1}' | gzip >$@

motif-background.bed.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/back-regions.txt.gz" | zcat | awk -vOFS='\t' '{print $$1, $$2 - 1, $$3}' | bedtools sort | bedtools merge | gzip >$@

.DELETE_ON_ERROR:
