CURL := curl -sfS

go-basic.obo.gz:
	$(CURL) -L "http://purl.obolibrary.org/obo/go/go-basic.obo" | gzip >$@

.DELETE_ON_ERROR:
