DATA := $(abspath ../../data)
CASES := BD CAD CD HT RA T1D T2D
COHORT := 58C NBS $(CASES)
AUTOSOMES := 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22

%-install:
	mv $**.gen.gz $(DATA)/wtccc/1kg-phase1

%-gz:
	parallel "cat $*_{}.*.gen | gzip >$*_{}.gen.gz" ::: $(AUTOSOMES)

%-concordance:
	awk '/>= 0.0/ {s += $$NF; n += 1} END {print s / n}' $**_summary

.PHONY: %-concordance %-gz %-install

joblist: $(foreach c,$(COHORT),$(foreach a,$(AUTOSOMES),$(c)_$(a).joblist))
	cat $^ >$@

%.joblist: C = $(shell echo $* | sed s/^.*_0\\?//)
%.joblist: $(DATA)/wtccc/haplotypes/%.haps.gz %.windows %.exclude
	parallel -C' ' --dry-run impute2 -seed 0 -use_prephased_g -known_haps_g $< -exclude_snps_g $*.exclude -impute_excluded -m $(DATA)/1kg/genetic_map_chr$(C)_combined_b37.txt -h $(DATA)/1kg/ALL.chr$(C).integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.haplotypes.gz -l $(DATA)/1kg/ALL.chr$(C).integrated_phase1_v3.20101123.snps_indels_svs.genotypes.nosing.legend.gz -int {1} {2} -filt_rules_l '"type==SV"' '"eur.maf<0.01"' -o $*.{#}.gen '>/dev/null' :::: $*.windows >$@

define EXCLUDE
import sys
ambiguous = ('AT', 'TA', 'CG', 'GC')
for line in sys.stdin:
    _, probe, _, a0, a1, *_ = line.split()
    if a0 + a1 in ambiguous:
        print(probe)
endef
export EXCLUDE

%-clean.sample: $(DATA)/wtccc/haplotypes/%.exclude $(DATA)/wtccc/haplotypes/%.sample
	awk 'FNR == NR {drop[$$2] = 1} $$2 in drop {next} FNR < NR {print}' $^ >$@

%.exclude: $(DATA)/wtccc/haplotypes/%.haps.gz
	zcat $< | python -c "$$EXCLUDE" >$@

.PRECIOUS: %.exclude

define FILTER
import csv, itertools, sys
window_size = 5000000
pos = (int(line.split()[2]) for line in sys.stdin)
for k, g in itertools.groupby(pos, key=lambda x: x // window_size):
    if len(list(g)) > 100:
        print('{} {}'.format(k * window_size, (k + 1) * window_size - 1))
endef
export FILTER

%.windows: $(DATA)/wtccc/haplotypes/%.haps.gz
	zcat $< | python -c "$$FILTER" >$@

clean:
	find \( -name "impute2.*" -o -name "*.log" -o -name "*.gen" -o -name "*.gen_*" \) | parallel -X rm
