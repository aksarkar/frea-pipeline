EC21 := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21
DATA := /broad/compbio/aksarkar/data
PROP := daf tss-abs-dist ld-partners-0.8
CODE := $(HOME)/code/struct/snp
export LC_ALL=C

all: $(PROP:=-by-cluster.pdf)

%-by-cluster.pdf: %-by-cluster.txt.gz
	Rscript $(CODE)/plot.R $< $(subst -,.,$*.plot)

%-by-cluster.txt.gz: 1.snps %.txt.gz
	parallel 'join {} <(zcat $*.txt.gz) | sed "s/^.* /{.} /"' ':::' *.snps | gzip >$@

table.txt.gz: maf.txt.gz tss-abs-dist.txt.gz ld-partners.txt.gz
	bash -c 'join <(zcat $(word 1, $^)) <(zcat $(word 2, $^)) | join - <(zcat $(word 3, $^)) | gzip' >$@

daf.txt.gz:
	zcat /broad/compbio/pouyak/motifs/verts/data/raw/var/1000genomes/201203/*.vcf.gz | awk -f $(CODE)/daf.awk | cut -f4,5 | sort -k1 | gzip >$@

maf.txt.gz:
	parallel -kj1 "zcat $(DATA)/1kg/ALL.chr{}.*.legend.gz | awk -vc={} -f $(CODE)/maf.awk" ":::" $(shell seq 1 22 | sort) | sort -k1 | gzip >$@

tss-abs-dist.txt.gz:
	bedtools closest -d -a $(DATA)/1kg/1kg-eur.bed.gz -b $(DATA)/encode/Gencodev10_CAGE_TSS_clusters_May2012.gff.gz | awk -vOFS='\t' '{print $$4, int($$NF/1000)}' | sort -k1 | gzip >$@

ld-score.txt.gz:
	zcat /broad/compbio/pouyak/popgen/proj/1kg/phase1/corr/EUR.txt.gz | python ~/code/struct/snp/ldscore.py | gzip >$@

ld-partners.txt.gz:
	zcat /broad/compbio/pouyak/popgen/proj/1kg/phase1/corr/EUR.txt.gz | python ~/code/struct/snp/ld_blocksize.py | gzip >$@

1.snps:
	parallel 'bedtools intersect -sorted -a /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/bed/t1d.wald.bed.gz -b {} | cut -f4 | sort -k1 >$$(basename {} .bed.gz).snps' ':::' $(EC21)/data/roadmap/hb-features/EnhClusters/*.gz

.PRECIOUS: %-by-cluster.txt.gz

.DELETE_ON_ERROR:
