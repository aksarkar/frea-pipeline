CURL := curl -sfSO

RA_GWASmeta2_20090505-results.txt:
	$(CURL) "https://www.broadinstitute.org/ftp/pub/rheumatoid_arthritis/Stahl_etal_2010NG/RA_GWASmeta2_20090505-results.txt"

pgc.bip.2012-04.zip:
	$(CURL) "http://www.med.unc.edu/pgc/files/resultfiles/pgc.bip.2012-04.zip"

scz2.snp.results.txt.gz:
	$(CURL) "http://www.med.unc.edu/pgc/files/resultfiles/scz2.snp.results.txt.gz"

pgc-scz-summary.txt.gz: scz2.snp.results.txt.gz
	zcat $< | awk 'NR == 1 {print; next} {print | "sort -k1,1 -k5,5n"}' | gzip >$@

iibdgc-trans-ancestry-summary-stats.tar:
	$(CURL) --ftp-pasv 'ftp://ftp.sanger.ac.uk/pub/consortia/ibdgenetics/iibdgc-trans-ancestry-summary-stats.tar'

igap.zip:
	false

diagram.zip:
	$(CURL:O=) -d confirm=1 "http://diagram-consortium.org/check_check.php"

cardiogram_gwas_results.zip:
	$(CURL) --ftp-pasv "ftp://ftp.sanger.ac.uk/pub/cardiogramplusc4d/cardiogram_gwas_results.zip"

hg19_gwas_t1d_bradfield_4_18_0.gz:
	$(CURL) "https://www.t1dbase.org/downloads/protected_data/GWAS_Data/hg19_gwas_t1d_bradfield_4_18_0.gz"

reprogen-menarche.bed.gz: Menarche_19052016.txt.gz
	python -c "from frea.summary.process import *; menarche('$<')" | gzip >$@

reprogen-menopause.bed.gz: Menopause_22062016.txt.gz
	python -c "from frea.summary.process import *; menopause('$<')" | gzip >$@
