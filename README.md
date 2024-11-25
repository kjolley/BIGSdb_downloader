# BIGSdb_downloader
Download alleles and profiles from PubMLST and BIGSdb 
Pasteur via their API using authentication.

Use in place of `wget` or `curl` in download scripts to seamlessly handle the
OAuth authentication required by the BIGSdb API.

# Installation
Download using git clone:

```
git clone https://github.com/kjolley/BIGSdb_downloader.git
```

Then install the single dependency:

```
pip install rauth
```
# Accessing PubMLST and BIGSdb Pasteur APIs using authentication
The BIGSdb platform used for PubMLST and BIGSdb Pasteur uses OAuth 
authentication that enables you to delegate access using your account to a
script without having to share credentials.

First you need to register an account for the appropriate site (see 
[https://pubmlst.org/site-accounts](https://pubmlst.org/site-accounts)).

The addresses you need to do this are:

* PubMLST: [https://pubmlst.org/bigsdb](https://pubmlst.org/bigsdb)
* Pasteur: [https://bigsdb.pasteur.fr/cgi-bin/bigsdb/bigsdb.pl](https://bigsdb.pasteur.fr/cgi-bin/bigsdb/bigsdb.pl)

You then need to register this account with each database that you want to 
access. This can also be done at the above addresses.

Finally, you will need to obtain a client key and secret. For PubMLST, you can
now create a personal key at [https://pubmlst.org/bigsdb](https://pubmlst.org/bigsdb). 
For Pasteur, you currently need to request this via an E-mail to the following 
address (but an automated method to obtain personal keys will be available 
soon):

* Pasteur - [bigsdb@pasteur.fr](mailto:bigsdb@pasteur.fr)

# Credential setup
The script will handle multiple keys for each site if necessary - you can call
these what you like but normally you would just have one that you can name the
same as the site. To set up the credentials for the first time run with the
--setup option and provide the name of a database configuration that your 
account has access to, e.g.

```
./bigsdb_downloader.py --key_name PubMLST --site PubMLST --db pubmlst_neisseria_isolates --setup
```
This will then prompt you to enter the client key and client secret that you 
have obtained. These will be stored in the token_directory
(./bigsdb_tokens by default but can be set using --token_dir argument).

You will then be prompted to login to a particular page on the BIGSdb site and
authorize delegation of your account access. This will provide you with a 
verification code that you will be prompted to enter by the script. Once done
an access token will be saved that will be used for all future access.

Session tokens will be obtained and renewed automatically by the script as 
required using your client key and access token.

# Downloading data
To download you would run something like the following:

```
./bigsdb_downloader.py --key_name PubMLST --site PubMLST --url "https://rest.pubmlst.org/db/pubmlst_neisseria_seqdef/schemes/1/profiles_csv"
```

It is also possible to use HTTP POST method calls and include a JSON payload.
For example, to perform a BLAST query of a single abcZ sequence against the
MLST scheme in the Neisseria database you can do:

```
./bigsdb_downloader.py --key_name PubMLST --site PubMLST --url "https://rest.pubmlst.org/db/pubmlst_neisseria_seqdef/schemes/1/sequence" --method POST --json_body '{"sequence":"TTTGATACCGTTGCCGAAGGTTTGGGCGAAATTCGTGATTTATTGCGCCGTTATCATCATGTCAGCCATGAGTTGGAAAATGGTTCGAGTGAGGCTTTGTTGAAAGAACTCAACGAATTGCAACTTGAAATCGAAGCGAAGGACGGCTGGAAACTGGATGCGGCAGTCAAGCAGACTTTGGGGGAACTCGGTTTGCCGGAAAATGAAAAAATCGGCAACCTTTCCGGCGGTCAGAAAAAGCGCGTCGCCTTGGCTCAGGCTTGGGTGCAAAAGCCCGACGTATTGCTGCTGGACGAGCCGACCAACCATTTGGATATCGACGCGATTATTTGGCTGGAAAATCTGCTCAAAGCGTTTGAAGGCAGCTTGGTTGTGATTACCCACGACCGCCGTTTTTTGGACAATATCGCCACGCGGATTGTCGAACTCGATC"}'
```
For payloads larger than the command line character limit, e.g. whole genome 
assemblies, you can write the JSON payload to a temporary file and pass this
filename as an option. For example to query a FASTA file, contigs.fasta, 
against the same scheme you can do:

```
file_contents=$(base64 -w 0 contigs.fasta)
json_body=$(echo -n '{"base64":true,"details":false,"sequence": "'; echo -n "$file_contents"; echo '"}')
echo "$json_body" > temp.json
./bigsdb_downloader.py --key_name PubMLST --site PubMLST --url "https://rest.pubmlst.org/db/pubmlst_neisseria_seqdef/schemes/1/sequence" --method POST --json_body_file temp.json 
```
# Options

```
./bigsdb_downloader.py --help
usage: bigsdb_downloader.py [-h] [--cron] [--db DB] [--json_body JSON_BODY] [--json_body_file JSON_BODY_FILE] --key_name
                            KEY_NAME [--method {GET,POST}] [--output_file OUTPUT_FILE] [--setup]
                            [--site {PubMLST,Pasteur}] [--token_dir TOKEN_DIR] [--url URL]

options:
  -h, --help            show this help message and exit
  --cron                Script is being run as a CRON job or non-interactively.
  --db DB               Database config - only needed for setup.
  --json_body JSON_BODY
                        JSON body to be included in a POST call. If this is longer than the command line limit (probably
                        about 128kb) then you will need to save the JSON payload to a file and use --json_body_file
  --json_body_file JSON_BODY_FILE
                        File containing JSON to use in the body of a POST call.
  --key_name KEY_NAME   Name of API key - use a different name for each site.
  --method {GET,POST}   HTTP method
  --output_file OUTPUT_FILE
                        Path and filename of saved file. Output sent to STDOUT if not specified.
  --setup               Initial setup to obtain access token.
  --site {PubMLST,Pasteur}
  --token_dir TOKEN_DIR
                        Directory into which keys and tokens will be saved.
  --url URL             URL for API call.
```

# API documentation
You can find details about all the API routes that you can call at 
[https://bigsdb.readthedocs.io/en/latest/rest.html](https://bigsdb.readthedocs.io/en/latest/rest.html).
