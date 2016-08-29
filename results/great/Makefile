DATA := $(abspath ../../data)
RESULTS := $(abspath ..)
SHELL := bash
MAKEOVERRIDES := "-j5"
INTERSECT := bedtools intersect -sorted -nobuf

PHENO := bradfield-t1d
THRESH := 3
CLUSTER := 1
FDR := .017

define FILTER
awk -vFS='\t' -vOFS='\t' 'FNR > 5 && $$1 == "GO Biological Process" && $$7 < $(FDR) && $$8 > 1 {print $$3, $$19}' $$*
endef
export FILTER

all: $(PHENO).$(CLUSTER).out

summary: $(addsuffix .summary,ad bip cad cd ra scz t1d t2d)

%.summary:
	bash -c "python -m frea.go $(DATA)/go/go-basic.obo.gz <(cat *-$*.*.out) >$@"

$(PHENO).$(CLUSTER).out: $(PHENO).$(CLUSTER).txt
	bash -c "$$FILTER $<" >$@

$(PHENO).%.txt: $(PHENO).%.bed.gz
	curl -fsSG "http://bejerano.stanford.edu/great/public/cgi-bin/greatStart.php" -d requestURL="http://web.mit.edu/aksarkar/great/$<" -d bgURL="http://web.mit.edu/aksarkar/EnhClusters/$*.bed.gz" -d requestSpecies=hg19 -d outputType=batch -o $@

bed: $(PHENO).$(CLUSTER).bed.gz

$(PHENO).%.bed.gz:
	zcat $(RESULTS)/matched/$(PHENO).bed.gz | awk '$$5 > $(THRESH)' | $(INTERSECT) -wa -u -a $(DATA)/roadmap/hb-features/EnhClusters/$*.bed.gz -b stdin | gzip >$@

clean:
	rm -f *.bed.gz *.txt *.out

joblist.1: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} :::: $< >$@

joblist: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} bed :::: $< >$@

clusters: ../matched/hb-features/EnhClusters-significant.in
	awk '{print $$1, $$3, $$4}' $< | sort -k1,1 -k2,2n >$@

new-genes.txt: genes
	parallel 'awk -vFS="\t" '"'"'$$2 != "" {if (s != "") {s = s","} s = s $$2} END {print "{.}", s}'"'" {} ::: *.genes >$@

known-genes.txt: genes
	parallel 'awk -vFS="\t" '"'"'$$3 != "" {if (s != "") {s = s","} s = s $$3} END {print "{.}", s}'"'" {} ::: *.genes >$@

genes:
	parallel -j1 -C'\t' make -B NAME={1} {2}.genes ::: "Alz	igap-ad" "Bip	pgc-bip" "Coronary	cardiogram-cad" "Crohn	iibdgc-cd" "Rheum	stahl-ra" "Schiz	pgc-scz" "Diabetes Mellitus, Type 1	bradfield-t1d" "Diabetes Mellitus, Type 2	diagram-t2d"	"Menarche	reprogen-menarche" "Menopause	reprogen-menopause"

%.genes:
	comm <(zgrep "$(NAME)" $(DATA)/phegeni/phegeni.txt.gz | cut -f7 | tr ' ' '\n' | sort | uniq) <(cut -f2 $*.*.out | tr ',' '\n' | sort | uniq) >$@

.PHONY: genes %.summary count-pathways

.PRECIOUS: %.bed.gz %.txt

.DELETE_ON_ERROR:
