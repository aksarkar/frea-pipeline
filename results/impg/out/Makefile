DATA := $(abspath ../../../data)

joblist:
	parallel --dry-run ImpG-Summary -p {.} -m ../maps/{/.}.map -t ../in/{/.}.map.typed -o {/.}.txt ::: ../beta/*.beta >$@

%.bed.gz:
	parallel -j1 python -m frea.summary.process_impg {} $(DATA)/1kg ::: $$(seq 1 22) | bedtools sort | gzip >$@

