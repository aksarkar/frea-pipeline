SHELL := bash
MAKEOVERRIDES := "-j5"
INTERSECT := bedtools intersect -sorted -nobuf
DATA := /broad/compbio/aksarkar/data

PHENO := bradfield-t1d
THRESH := 3
CLUSTER := 1
FDR := .05

%.summary:
	awk '{print $5}' *-$*.*.out | tr ',' '\n' | sort | uniq | wc -l
	head *-$*.*.out

.PHONY: %.summary

$(PHENO).$(CLUSTER).out: $(PHENO).$(CLUSTER).txt
	awk -f ~/code/enr/great/post.awk $< >$@

$(PHENO).%.txt: $(PHENO).%.bed.gz
	curl -fsSG "http://bejerano.stanford.edu/great/public/cgi-bin/greatStart.php" -d requestURL="http://web.mit.edu/aksarkar/great/$<" -d bgURL="http://web.mit.edu/aksarkar/EnhClusters/$*.bed.gz" -d requestSpecies=hg19 -d outputType=batch -o $@

bed: $(PHENO).$(CLUSTER).bed.gz

$(PHENO).%.bed.gz:
	zcat /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/matched/core-features/$(PHENO).bed.gz | awk '$$5 > $(THRESH)' | $(INTERSECT) -wa -u -a $(DATA)/roadmap/hb-features/EnhClusters/$*.bed.gz -b stdin | gzip >$@

clean:
	rm -f *.bed.gz *.txt *.out

joblist.1: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} :::: $< >$@

joblist: clusters
	parallel --dry-run -C" " make PHENO={1} CLUSTER={2} THRESH={3} bed :::: $< >$@

clusters: ../matched/hb-features/EnhClusters.in
	awk '$$4 > 0' $< | sort -k8n | awk 't == 0 && $$8 > NR * $(FDR) / (226 * 8) {t = $$8} t == 0 || $$8 <= t {print $$1, $$3, $$4}' | sort -k1,1 -k2,2n >$@

$(PHENO).pathways.txt: $(CLUSTERS:=.out)
	cat $^ | sort -k1 | join -t$$'\t' - $(DATA)/roadmap/meta/manolis-clusters.txt -o "2.1 2.2 1.2 1.3 1.4 1.5 1.6" | sort -k1,1n -k6,6g -k4,4gr | awk -vOFS='\t' 'BEGIN {print "cluster", "description", "term", "fold", "p", "q", "genes"} {print}' >$@

candidates: pathways.txt
	sed -E -n "s/.*Name=([^;]*);.*/\1/p" $(DATA)/mask/t1d.genes.gff | sort | uniq | parallel -j1 grep -o -m1 {} $< >$@

new-genes: genes
	parallel 'awk -vFS="\t" '"'"'$$2 != "" {if (s != "") {s = s","} s = s $$2} END {print "{.}", s}'"'" {} ::: *.genes >$@

known-genes: genes
	parallel 'awk -vFS="\t" '"'"'$$3 != "" {if (s != "") {s = s","} s = s $$3} END {print "{.}", s}'"'" {} ::: *.genes >$@

genes:
	parallel -j1 -C'\t' make NAME={1} {2}.genes ::: "Alz	igap-ad" "Bip	pgc-bip" "Coronary	cardiogram-cad" "Crohn	iibdgc-cd" "Rheum	stahl-ra" "Schiz	pgc-scz" "Diabetes Mellitus, Type 1	bradfield-t1d" "Diabetes Mellitus, Type 2	diagram-t2d"

.PHONY: genes

%.genes:
	comm <(grep "$(NAME)" $(DATA)/phegeni/phegeni.txt | cut -f7 | tr ' ' '\n' | sort | uniq) <(cut -f6 $*.*.out | tr ',' '\n' | sort | uniq) >$@

.PRECIOUS: %.bed.gz %.txt

.DELETE_ON_ERROR:
