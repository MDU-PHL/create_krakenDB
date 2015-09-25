# create_krakenDB: a script to create an MDU-specific kraken DB

The MDU `kraken` DB should include *bacteria*, *virus*, *fungi*, and *synthetic
sequences* (e.g., Illumina adaptors). At some point, we will require also
*parasites*.

## Dependencies

	* Latest taxonomy from NCBI
	* Genomic sequences from the aformentioned organisms
	* A FASTA file with adaptor sequences formatted with the *synthetic DNA*
	  NCBI code (details below)

## Algorithm outline
How will this script accomplish its goal?

	1. Download taxonomy
	   
	   Sample command:
	   > kraken-build --download-taxonomy --db $DBNAME

	2. Add sequences to it
	   Sample command:
	   > kraken-build --add-to-library chr1.fa --db $DBNAME
	
		* Assumption 1: Sequences are in FASTA file
		* Assumption 2: The header includes GI number so kraken can
				match up to NCBI's taxonomic database
		* Assumption 3: If assumption 2 is not met, then the sequence
				header includes the karken taxon-id
		* Assumption 4: Unclear if this step can be done in parallel

	3. Build the database
	   Sample command:
	   > kraken-build --build --db $DBNAME

		* Assumption 1: The default command assumes kmer length 31, it is
 				not clear to me at the outset if this is suitable
				for MDU's purposes
		* Assumption 2: The default minimiser length is 12. This is a 
				parameter that will mostly affect speed of search.
				But, the exact effect of a value (faster/slower)
				queries must be empirically determined.
		* Assumption 3: Maximum size of the DB. We can set a maximum size,
				which once reached, will cause Kraken to subsample
				kmer:taxon pairs to fit within the limits. Or, allow
				it to be as big as it needs to be. The default is to
				let it grow as much as it needs to.
		* Assumption 4: We can and should parallelise this step. I am thinking
				at 36 threads?

	4. OPTIONAL Thin the database
	   Sample command:
	   > kraken-build --shrink 10000 --db $DBNAME --new-db minikraken

		* The above command would make the DB exactly 10000 kmer:taxon
		  pairs. Seems small. Here, a decision can be later made. However,
		  presumably the smaller the database, the faster the search. So,
		  There is likely some room here for optimisation. Unclear how much
		  time we would gain by going from 1M kmer:taxon pairs to 100K or 
	   	  less, and what that would mean for our accuracy.

## Things we will need

Adding sequences to the DB is the most problematic step.

Our data is stored in \*.gbk.gz files. This holds two problems:

	1. First none of the kraken tools read compressed files
	2. Second none of the kraken tools read Genbank files

Solving the first one is relatively straightforward. A call to `gunzip` might suffice.
The second step is deceiving. While there are tools readily available that will 
easily convert `genbank` format into `fasta` format, I have not found one yet that 
will correctly add the **GI** number to the header that is necessary for `kraken`
to correctly parse the taxon information associated with the sequence. The solution 
might require writing a `Python` or `PERL` script to wrap around the whole of 
*step 2* of our algorithm. This might allow us to take advantage of some parallelism
by adding suitable semaphores. 
