CURL := curl -sfS

define PARSE
function clear() {
    motif = ""
    pwm = ""
    n = 0
    cutoff = 0
}

function output() {
    if (motif != "") {
        for (i=0; i < n; i++) {
            printf "X"
        }
        printf "\t%s\n%lf%s\n", motif, cutoff, pwm
    }
}

/^[A-Z]/ {
    if (motif != "") {
        pwm = pwm "\n" $$0
        n += 1
    }
}

/^>/ {
    output()
    clear()
    sub(">", "", $$1)
    if ($$1 !~ /_(disc|C)/) {
        motif = $$1
        cutoff = $$2
    }
}
endef
export PARSE

all: clusters.txt $(addsuffix .txt.gz,enrichments expr-corr motifs-toscan) $(addsuffix .bed.gz,matches motif-background)

clusters.txt: motifs-clust.txt
	awk '{split($$2, a, "[(),]*"); for (i in a) {if (a[i] != "") {print a[i], $$1}}}' $< >$@

enrichments.txt.gz:
	$(CURL) -O "http://egg2.wustl.edu/roadmap/data/byDataType/motifanalysis/pouyak/clustered/enrichments.txt.gz"

expr-corr.txt.gz:
	$(CURL) -O "http://egg2.wustl.edu/roadmap/data/byDataType/motifanalysis/pouyak/clustered/expr-corr.txt.gz"

motifs-toscan.txt.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/motifs-toscan.txt.gz" | zcat | awk "$$PARSE" | gzip >$@

matches.bed.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/matches.txt.gz" | zgrep -v "_C[0-9]" | awk -vOFS="\t" '{sub(/_8mer/, "", $$1); print $$2, $$3-1, $$4, $$1}' | gzip >$@

motif-background.bed.gz:
	$(CURL) "http://compbio.mit.edu/encode-motifs/back-regions.txt.gz" | zcat | awk -vOFS='\t' '{print $$1, $$2 - 1, $$3}' | bedtools sort | bedtools merge | gzip >$@

.DELETE_ON_ERROR:
