PHENO := bradfield-t1d
THRESH := 3
CLUSTER := 1
FDR := 0.05

DATA := $(abspath ../../../data)
ENCODE := $(DATA)/encode
MHC := $(DATA)/mask/mhc+5mb.bed.gz
GWAS := ../../impg/out
ENHCLUSTERS := $(DATA)/roadmap/hb-features/EnhClusters

INTERSECT = bedtools intersect -sorted -nobuf

define KNOWN
awk -vOFS='\t' -vFS='\t' 'FNR == NR {sub(/_.*$$/, "", $$4); tf[$$4] = 1} FNR < NR {split($$7, genes, " "); for (g in genes) {if (genes[g] in tf) {print $$2, genes[g]}}}' <(zcat $< | tr ' ' '\t') <(zcat $(word 2, $^)) | sort -t$$'\t' -k1 | uniq
endef
export KNOWN

export LC_ALL := C
export PATH := $(PATH):/broad/compbio/pouyak/local/bin
export TMPDIR := /broad/hptmp/aksarkar

all: $(addprefix $(PHENO).$(CLUSTER).,fisher.txt summary matches)

%.pdf: %.dot
	circo -x -Tpdf -o$@ $<

all-enrichments-network.dot:
	python -m frea.motifs.network >$@

known-tfs.txt: all-enrichments-unclustered.txt.gz $(DATA)/phegeni/phegeni.txt.gz
	bash -c "$$KNOWN" >$@

all-enrichments-unclustered.txt.gz:
	python -c "from frea.motifs.enrichments import *; output_cofactors()" | gzip >$@


all-enrichments-summary.txt.gz:
	awk 'FNR == 1 {split(FILENAME, a, ".")} {print a[1], a[2], $$0}' *.fisher.txt | sort -k1,1 -k2,2n -k5,5g | gzip >$@

all-matches.txt.gz:
	awk 'FNR == 1 {split(FILENAME, a, ".")} {print a[1], a[2], $$0}' *.matches | gzip >$@

regulators: $(foreach p,$(wildcard $(GWAS)/*.bed.gz),$(notdir $(p:.bed.gz=.regulators)))

%.regulators:
	cat $*.*.summary | awk 'function i() {k=$$1; sub(/_[0-9]$$/, "", $$2); m=$$2} NR == 1 {i()} $$1 != k {print m; i()} END {print m}' | sort | uniq | awk 'NR == 1 {r = $$0} NR > 1 && NR < 6 {r = r "," $$0} END {if (NR >= 6) {r = r " (" NR " total)"}; print "$*", r}'

$(PHENO).%.matches: $(PHENO).%.fisher.txt $(PHENO).top.bed.gz c%.bed.gz
	awk 'NF == 4 {print $$1}' $< | parallel -j1 --halt now,fail=1 "zgrep {} c$*.bed.gz | $(INTERSECT) -u -wa -a $(ENHCLUSTERS)/$*.bed.gz -b stdin | $(INTERSECT) -wa -wb -a $(PHENO).top.bed.gz -b stdin | frea-motifs {} $(ENCODE)/motifs-toscan.txt.gz" >$@

$(PHENO).%.summary: $(PHENO).%.fisher.txt
	awk 'NF == 4' $< | sort -k1 | bash -c "join <(sort -k1 $(ENCODE)/clusters.txt) - -o '1.2 2.1 2.2 2.3'" | sort -k1,1 -k4,4n >$@

$(PHENO).%.fisher.txt: $(addprefix $(PHENO).%.,$(addsuffix .counts,fg bg) $(addsuffix .regions,fg bg))
	python -m frea.motifs.fisher $^ $(FDR) >$@

$(PHENO).%.counts: $(PHENO).%.hits
	python -m frea.motifs.counts $(ENCODE)/insts.txt <$< >$@

$(PHENO).%.hits: $(PHENO).%.regions c$(CLUSTER).bed.gz 
	$(INTERSECT) -wa -wb -a $< -b c$(CLUSTER).bed.gz | cut -f1,2,3,7 >$@

$(PHENO).%.fg.regions: $(PHENO).%.bg.regions $(PHENO).top.bed.gz
	$(INTERSECT) -u -a $(word 1, $^) -b $(word 2, $^) >$@

$(PHENO).%.bg.regions:
	$(INTERSECT) -u -a $(ENHCLUSTERS)/$*.bed.gz -b $(ENCODE)/motif-background.bed.gz >$@

top: $(foreach p,$(wildcard $(GWAS)/*.bed.gz),$(notdir $(p:.bed.gz=.top.bed.gz)))

%.top.bed.gz:
	$(INTERSECT) -v -a $(GWAS)/$*.bed.gz -b $(DATA)/mask/mhc+5mb.bed.gz | awk '$$5 >= $(THRESH)' | gzip >$@

c%.bed.gz: enriched
	zcat $(ENCODE)/matches.bed.gz | awk 'BEGIN {while (getline < "$<") {if ($$4 == "c$*") {keep[$$2] = 1}}} keep[$$4] > 0 {print}' | $(INTERSECT) -u -a stdin -b $(ENHCLUSTERS)/$*.bed.gz | gzip >$@

enriched:
	zcat $(ENCODE)/enrichments.txt.gz | awk '$$14 > 1.5 {split($$1, a, "/"); if (a[4] > 0.3) {print}}' >$@

joblist.1: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} :::: $< >$@

joblist: clusters
	awk '{print $$2}' $< | sort | uniq | parallel --dry-run make c{}.bed.gz >$@

clusters: ../../matched/hb-features/EnhClusters-significant.in
	awk '{print $$1, $$3, $$4}' $< | sort -k1,1 -k2,2n >$@

clean:
	parallel -X rm -f ::: *.fg.* *.bg.* *.fisher.txt *.disruptions *.counts *.summary

.DELETE_ON_ERROR:

.PHONY: top clean
