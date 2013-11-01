# TYPO3 find configuration for Germania Sacra
This repository contains the configuration for the [TYPO3 find extension](https://github.com/subugoe/typo3-find) to display the Germania Sacra index.


## Setup
To set up the Germania Sacra index, you need

1. to have this repository as the `Projects/germania-sacra` subfolder inside the find extension

2. your TYPO3 set up with:

	1. a **page** for Germania Sacra
	2. the find extension’s **plug-in** added on that page
	3. the find extension’s **template** include added to that page
	4. the template of that page containing `<INCLUDE_TYPOSCRIPT: source="FILE:typo3conf/ext/find/Projects/germania-sacra/germania-sacra.ts">` to **load the TypoScript template configuration** provided in this repository

3. to have access to the Germania Sacra index (initially created from [germania-sacra-daten](https://github.com/subugoe/germania-sacra-daten/tree/master/klosterdatenbank_neu#indexierung)), and the template of your page containing the **configuration to access the Solr index** created from it, e.g.

		plugin.tx_find.settings.connection {
			path = /solr/germania-sacra
			host = my.solr.server
			port = 8080
		}

4. a page on your site with the Germania Sacra **bibliography** imported in the [bib extension](https://github.com/subugoe/typo3-bib) and the page ID of that page configured in the template of your page, e.g.

		plugin.tx_find.settings.tx_bib_pid = 135

5. for **access control**:

	1. a TYPO3 frontend user group named »Germania Sacra« set up for users who may see as-yet unpublished records

	2. add the following TypoScript condition to your template (with 1 replaced by the ID of the »Germania Sacra« frontend user group) to remove the record filter for project members:

			[usergroup = 1]
			plugin.tx_find.settings.additionalFilters.1 >
			[global]

6. for **nice URLs**:

	1. TYPO3 should be configured to serve the page at `http://klosterdatenbank.germania-sacra.de/`

	2. single records should be served at the path `/gsn/123/` where 123 is the ID (klosternummer) of the record; this is achieved by adding a RealURL configuration as follows for the page (where `_DEFAULT` should be replaced by the page ID for the TYPO3 page as the required »gsn« deviates from the auto-configured »id« the extension suggests).

			'postVarSets' => array(
				'_DEFAULT' => array(
					'gsn' => array(
						array(
						'GETvar' => 'tx_find_find[id]',
					)
				),
			)

7. to serve **linked data** formats based on Accept-headers a mod_rewrite configuration along the following lines can do the job in the site’s .htaccess file (untested for the host name):

		RewriteCond %{HTTP_ACCEPT} text/turtle
		RewriteCond %{HTTP_HOST} klosterdatenbank.germania-sacra.de
		RewriteRule .* ?type=1380124799&tx_find_find\%5Bformat\%5D=data&tx_find_find\%5Bdata-format\%5D=turtle

		RewriteCond %{HTTP_ACCEPT} application/rdf+xml
		RewriteCond %{HTTP_HOST} klosterdatenbank.germania-sacra.de
		RewriteRule .* ?type=1378891468&tx_find_find\%5Bformat\%5D=data&tx_find_find\%5Bdata-format\%5D=rdf&tx_find_find\%5Bdata-fields\%5D=*

		RewriteCond %{HTTP_ACCEPT} text/json
		RewriteCond %{HTTP_HOST} klosterdatenbank.germania-sacra.de
		RewriteRule .* ?type=1369315139&tx_find_find\%5Bformat\%5D=data&tx_find_find\%5Bdata-format\%5D=json-ld&tx_find_find\%5Bdata-fields\%5D=*

8. **known issue**: the AdW site’s RealURL configuration (as of 2013-11-01) will remove the `type` parameter for the custom page types configured by this setup. This breaks AJAX calls as well as download links.



### TypoScript configuration
The main configuration is done in the [germania-sacra.ts](germania-sacra.ts) file that is included in the template.

Please refer to that file and its comments for the details.



## Templates
There are custom Index and Detail templates in `Templates/Search`.

The data templates symlink to the default templates.


## Partials
Most subfolders with partials are symlinked to the defaults provided by the find extension. The custom partials are listed below.

### GS
Germania Sacra specific partials.

* Karte: creates the map for the detail view
* Orden: creates the list of orders the monastery belonged to
* Standorte: creates the list of locations the monastery was situated at
* Permalink: Returns the permanent URI for the record (.data and .html)
* Quelle: creates a link to the original source of data on the monastery
* Band: creates the information on the Germania Sacra volume containing the monastery in question, depending on the data that is available it may come with a page number, a volume link or a page link
* Links: creates a list of links related to the monastery
* Personen: creates the list of persons who were members of the monastery
* JavaScript: inserts germania-sacra.js and sets variables used to load data for the maps
* DownloadLinks: creates links for downloading CSV and BNA files with paging. Uses the DownloadLink partial

### Bib
Used to load bibliography data for the citekeys stored in the index and format it.

* Bibliography: creates the bibliography
* Bibitem: formats a single bibliography entry
* Authors: formats the author information of a bibliography entry

### Formats
In addition to the standard formats, the Germania Sacra setup provides output as CSV, BNA and linked data turtle, RDF and JSON-LD.

#### Export of standort-orden documents
These are linked for search results if the user is logged in:

* bna: creates BNA output
* csv: creates CSV output

#### Linked data output
Linked data output is returned if a linked data media type is requested in the http Accept header (see the .htaccess configuration above).

* linked-data: the main setup creating the triples from the documents; called by the following
* turtle: returns the triples in turtle format
* rdf: returns the triples in RDF format
* json-ld: returns the triples in JSON-LD format

Example linked data output in turtle format is available in the [Resources/linked-data-examples folder](Resources/linked-data-examples).


## Language
No real localisation is provided for the Germania Sacra setup as the interface (as well as the data) is availble in German only.

Still, some of the terms used in the user interface are provided in locallang XML files.

* locallang-facets: facet names
* locallang-form: form field labels and placeholders
* locallang: symlink to the extension’s default file


## Resources
A number of resources are included with this setup. They need to be accessed for the display to work correctly.

* Ordenssymbole: folder of PNG files with icons representing the different orders; these are included in the map display
* germania-sacra.css: included to style the output
	* this file is created from the germania-sacra.scss file and should not be edited directly
* germania-sacra.js: JavaScript for the map display
* bib.css: included to style the output
	* this file is created from the bib.scss file and should not be edited directly
* Bistumsgrenzen: This folder does not exist currently but used to contain a KML file with the diocese boundaries
