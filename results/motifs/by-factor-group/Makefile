DATA = /broad/compbio/aksarkar/data
ENCODE := $(DATA)/encode
CODE := $(HOME)/code/enr/motifs
INTERSECT = bedtools intersect -sorted -nobuf
MHC := $(DATA)/mask/mhc+5mb.bed.gz

PHENO := bradfield-t1d
GWAS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/out/$(PHENO).bed.gz
MOTIFS := $(ENCODE)/groups.txt
THRESH := 3
CLUSTER := 1
FDR := 0.05

all: $(PHENO).$(CLUSTER).fisher.txt $(PHENO).$(CLUSTER).disruptions

$(PHENO).%.disruptions: $(PHENO).%.cofactors.counts
	python -m frea.motifs.filter_disruptions $< /broad/compbio/pouyak/motifs/verts/data/motifs/tfh/table/motifs.txt | sort -k1,1 -k5,5gr >$@

$(PHENO).%.cofactors.counts: $(PHENO).%.fisher.txt $(PHENO).%.fg.hits $(PHENO).top.bed.gz
	awk 'NF == 4 {print $$1}' $(word 1, $^) | parallel --halt now,fail=1 "grep {} $(word 2, $^) | $(INTERSECT) -a $(ENCODE)/matches.bed.gz -b stdin | $(INTERSECT) -wb -a $(word 3, $^) -b stdin | python -m frea.motifs.count_disruptions {}" >$@

$(PHENO).%.fisher.txt: $(addprefix $(PHENO).%.,$(addsuffix .counts,fg bg) $(addsuffix .regions,fg bg))
	python -m frea.motifs.fisher $^ $(FDR) >$@

$(PHENO).%.counts: $(PHENO).%.hits
	python -m frea.motifs.counts $(ENCODE)/groups.txt <$< >$@

$(PHENO).%.hits: $(PHENO).%.regions
	$(INTERSECT) -wa -wb -a $< -b $(ENCODE)/matches.bed.gz | cut -f1,2,3,7 >$@

$(PHENO).%.fg.regions: $(PHENO).%.bg.regions $(PHENO).top.bed.gz
	$(INTERSECT) -u -a $(word 1, $^) -b $(word 2, $^) >$@

$(PHENO).top.bed.gz:
	zcat $(GWAS) | awk '$$5 >= $(THRESH)' | gzip >$@

$(PHENO).%.bg.regions:
	$(INTERSECT) -u -a $(GWAS) -b $(ENCODE)/motif-background.bed.gz | $(INTERSECT) -u -a $(DATA)/roadmap/hb-features/EnhClusters/$*.bed.gz -b stdin >$@

joblist: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} :::: $< >$@

clusters: ../../matched/hb-features/EnhClusters.in
	awk '$$4 > 0' $< | sort -k8n | awk 't == 0 && $$8 > NR * $(FDR) / (226 * 8) {t = $$8} t == 0 || $$8 <= t {print $$1, $$3, $$4}' | sort -k1,1 -k2,2n >$@

.DELETE_ON_ERROR:
