WGET = wget -q -r -N -l0 --no-parent -P $@

all: core honeybadger $(addprefix honeybadger2-impute/p,2 10)

core:
	mkdir -p $@
	$(WGET) -A "E*mnemonics.*" "https://www.broadinstitute.org/~anshul/projects/roadmap/segmentations/models/coreMarks/parallel/set2/final/"

honeybadger:
	mkdir -p $@
	$(WGET) "https://www.broadinstitute.org/~meuleman/reg2map/HoneyBadger_release/enh/clustering_10kb/BED_files/"

honeybadger2-impute-p%:
	mkdir -p $@
	$(WGET) "https://www.broadinstitute.org/~meuleman/reg2map/HoneyBadger2-impute_release/DNase/p$*/enh/BED_files_per_cluster/"

.DELETE_ON_ERROR:
