WGET = wget -q -r -N -l0 --no-parent -P $@

all: core honeybadger

core:
	mkdir -p $@
	$(WGET) -A "E*mnemonics.*" "https://www.broadinstitute.org/~anshul/projects/roadmap/chromhmmSegmentations/ChmmModels/coreMarks/parallel/set2/final/"

honeybadger:
	mkdir -p $@
	$(WGET) "https://www.broadinstitute.org/~meuleman/reg2map/HoneyBadger_release/enh/clustering_10kb/BED_files/"

.DELETE_ON_ERROR:
