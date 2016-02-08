HAPS := /broad/compbio/aksarkar/projects/gwas/wtccc1/EC21/results/haplotypes

joblist:
	awk -v PREFIX=$(HAPS) -f /home/unix/aksarkar/code/wtccc/gcta/joblist.awk

.DELETE_ON_ERROR:

.PHONY: joblist
