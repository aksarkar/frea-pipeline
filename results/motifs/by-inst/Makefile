PHENO := bradfield-t1d
THRESH := 3
CLUSTER := 1
FDR := 0.05

DATA := /broad/compbio/aksarkar/data
ENCODE := $(DATA)/encode
MHC := $(DATA)/mask/mhc+5mb.bed.gz
GWAS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/out
ENHCLUSTERS := /broad/compbio/aksarkar/data/roadmap/hb-features/EnhClusters

INTERSECT = bedtools intersect -sorted -nobuf

define KNOWN
awk -vOFS='\t' -vFS='\t' 'FNR == NR {sub(/_.*$$/, "", $$2); tf[$$2] = 1} FNR < NR {if (tf[$$7]) {print $$2, $$7}}' <(cat *.summary | tr ' ' '\t') <(zcat $(DATA)/phegeni/phegeni.txt.gz) | sort -t$$'\t' -k1
endef
export KNOWN

export LC_ALL := C
export PATH := $(PATH):/broad/compbio/pouyak/local/bin
export TMPDIR := /broad/hptmp/aksarkar

all: $(addprefix $(PHENO).$(CLUSTER).,fisher.txt summary matches)

known-tfs.txt:
	bash -c "$$KNOWN" >$@

counts: $(foreach p,$(wildcard $(GWAS)/*.bed.gz),$(notdir $(p:.bed.gz=.counts)))

%.counts:
	cat $*.*.matches | wc -l

regulators: $(foreach p,$(wildcard $(GWAS)/*.bed.gz),$(notdir $(p:.bed.gz=.regulators)))

%.regulators:
	cat $*.*.summary | awk 'function i() {k=$$1; sub(/_[0-9]$$/, "", $$2); m=$$2} NR == 1 {i()} $$1 != k {print m; i()} END {print m}' | sort | uniq | awk 'NR == 1 {r = $$0} NR > 1 && NR < 6 {r = r "," $$0} END {if (NR >= 6) {r = r " (" NR " total)"}; print "$*", r}'

$(PHENO).%.matches: $(PHENO).%.fisher.txt $(PHENO).top.bed.gz c%.bed.gz
	awk 'NF == 4 {print $$1}' $< | parallel -j1 --halt now,fail=1 "zgrep {} c$*.bed.gz | $(INTERSECT) -u -wa -a $(ENHCLUSTERS)/$*.bed.gz -b stdin | $(INTERSECT) -wa -wb -a $(PHENO).top.bed.gz -b stdin | frea-motifs {} $(ENCODE)/motifs-toscan.txt >$@

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

clusters: ../../matched/hb-features/EnhClusters.in
	awk '$$4 > 0' $< | sort -k8n | awk 't == 0 && $$8 > NR * $(FDR) / (226 * 8) {t = $$8} t == 0 || $$8 <= t {print $$1, $$3, $$4}' | sort -k1,1 -k2,2n >$@

clean:
	parallel -X rm -f ::: *.fg.* *.bg.* *.fisher.txt *.disruptions *.counts *.summary

.DELETE_ON_ERROR:

.PHONY: top clean
