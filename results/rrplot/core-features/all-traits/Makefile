joblist:
	parallel --dry-run bash ~/code/enr/generic/bin/bin.sh -c -i /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/restrict/all-studies-impg.bed.gz -s /broad/compbio/aksarkar/data/mask/mhc+5mb.bed.gz {1} {2} ::: /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/out/*.bed.gz ::: /broad/compbio/aksarkar/data/roadmap/core-features/{Enh,Tss,Tx}/*.gz >joblist

joblist.1:
	parallel --dry-run Rscript /home/unix/aksarkar/code/enr/plot/rrplot.R {} 1 pdf rrplot_snps ::: *.in.gz >$@

cutoffs:
	fgrep -h [1] rrplot.o* | tr -d '"' | awk '{sub(/\..*in/, ".bed", $2); print $2, $3}'  | parallel -C" " -j1 'bedtools intersect -sorted -nobuf -wa -a /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/impg/out/{1} -b /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/restrict/all-studies-impg.bed.gz | bedtools intersect -sorted -nobuf -wa -v -a stdin -b /broad/compbio/aksarkar/data/mask/mhc+5mb.bed.gz | sort -k5gr | awk '"'"'NR == {2} {print "{1}", $5}'"'" >cutoffs
