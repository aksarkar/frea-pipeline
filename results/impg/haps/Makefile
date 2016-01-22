joblist.1:
	parallel --dry-run 'bash $HOME/code/impg/pre/split_haps.sh {} $$(basename {} | sed "s#\..*##").haps >{/.}.haps' ::: ../maps/*.map

joblist:
	parallel --dry-run "bash $HOME/code/impg/pre/haps.sh {} >{}.haps" ::: $$(seq 1 22)
