DATA := $(abspath ../../../data)

export LC_ALL=C

joblist.1:
	parallel --dry-run 'frea-split-haps $$(basename {} | sed "s#\..*##").haps.gz {} $(DATA)/1kg/ALL.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.sample >{/.}.haps' ::: ../maps/*.*.map >$@

joblist:
	parallel --dry-run "frea-convert-haps $(DATA)/1kg/ALL.chr{}.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.haplotypes.gz $(DATA)/1kg/ALL.chr{}.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.legend.gz | gzip >{}.haps.gz" ::: $$(seq 1 22) >$@
