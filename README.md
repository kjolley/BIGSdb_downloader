# BIGSdb_downloader
Download alleles and profiles from PubMLST and BIGSdb 
Pasteur via their API using authentication.

Use in place of `wget` or `curl` in download scripts to seamlessly handle the
OAuth authentication required by the BIGSdb API.

# Installation
Install using git clone:

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

Finally, you will need to obtain a client key and secret. Currently you need to
request this via an E-mail to the following addresses (but an automated method
to obtain personal keys will be available soon):

* PubMLST - [pubmlst@biology.ox.ac.uk](mailto:pubmlst@biology.ox.ac.uk)
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

To download you would run something like the following:

```
./bigsdb_downloader.py --key_name PubMLST --site PubMLST --url "https://rest.pubmlst.org/db/pubmlst_neisseria_seqdef/schemes/1/profiles_csv"
```

# Options

```
./bigsdb_downloader.py --help
usage: bigsdb_downloader.py [-h] [--cron] [--db DB] --key_name KEY_NAME [--output_file OUTPUT_FILE] [--setup]
                            [--site {PubMLST,Pasteur}] [--token_dir TOKEN_DIR] [--url URL]

options:
  -h, --help            show this help message and exit
  --cron                Script is being run as a CRON job or non-interactively.
  --db DB               Database config - only needed for setup.
  --key_name KEY_NAME   Name of API key - use a different name for each site.
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