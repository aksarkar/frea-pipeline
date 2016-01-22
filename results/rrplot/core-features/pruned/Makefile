all: $(foreach r,1 8,0.$(r)/joblist)

%/joblist:
	mkdir -p $*
	parallel --dry-run bash ~/code/enr/generic/bin/bin.sh -c -l weight -i /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/restrict/all-studies-impg.bed.gz -s /broad/compbio/aksarkar/data/mask/mhc+5mb.bed.gz {1} {2} 100 ::: /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/ld/pruned/$*/*.bed.gz ::: /broad/compbio/aksarkar/data/roadmap/core-features/{Enh,Tss,Tx}/*.bed.gz >$@
