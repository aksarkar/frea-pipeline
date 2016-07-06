CURL = curl -sfSO "http://egg2.wustl.edu/roadmap/data/byDataType/rna/expression/$@"

all: Ensembl_v65.Gencode_v10.ENSG.gene_info 57epigenomes.RPKM.pc.gz EG.name.txt

Ensembl_v65.Gencode_v10.ENSG.gene_info:
	$(CURL)

57epigenomes.RPKM.pc.gz:
	$(CURL)

EG.name.txt:
	$(CURL)
