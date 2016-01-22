1KG := /broad/compbio/aksarkar/data/1kg

joblist.1:
	parallel --dry-run GenMaps -m {}.map -p {} -w 5000000 -s {}.names ::: $$(seq 1 22)

joblist:
	parallel --dry-run "python -m frea.impg.map $(1KG)/ALL.chr{}.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.legend.gz >{}.map" ::: $$(seq 1 22)
