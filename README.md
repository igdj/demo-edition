Demo Web-site based on TeiEditionBundle
=======================================

This is an edition for documentation purposes demonstrating the use of
    https://github.com/igdj/tei-edition-bundle/

You may use it in parts or adjust it to your own need if it fits your needs.
If you have any questions or find this code helpful, please contact us at
    http://jewish-history-online.net/contact

Installation Notes
------------------
Requirements
- PHP 7.3 or 7.4 (check with `php -v`)
  PHP 8 doesn't work (due to "solarium/solarium": "^5.1")
- composer (check with `composer -v`; if it is missing, see https://getcomposer.org/)
- MySQL or MariaDB (for metadata storage)
- Java 1.8 (for XSLT and Solr, check with `java -version`)
- `convert` (for image tiles, check with `which convert`; if it is missing, install e.g. with `sudo apt-get install imagemagick`)

In a fitting directory (e.g. `/var/www`), clone the project

    git clone https://github.com/igdj/demo-edition.git

If you don't have `git` installed, you can also download the project as ZIP-file
and extract it manually.

Change into the newly created project-directory

    cd demo-edition

Install dependencies

    composer install

Create database

    mysqladmin -u root -p create demo_edition

and create a database user with proper rights, e.g.

    mysql -u root -p demo_edition

Create a user and grant the needed privileges

    CREATE USER 'demo_edition'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD';
    GRANT ALL ON demo_edition.* TO 'demo_edition'@'localhost';

Create your local settings

    cp config/parameters.yaml-dist config/parameters.yaml

In `config/parameters.yaml`, adjust the database settings as by the
database, user and password set above:
    `database.name` / `database.user` / `database.password`)

Make `bin/console` executable

    chmod u+x ./bin/console

Alternatively, you can prepend to `./bin/console` in what follows

    php ./bin/console help

Create the database tables

    ./bin/console doctrine:schema:create

### XSLT-Processor
If you don't have Saxon/C (https://www.saxonica.com/saxon-c/index.xml)
installed as a PHP-module in your web server (which is quite tricky),
you can use the command line adapter.

For this, download `saxon9he.jar` as part of `SaxonHE9-9-1-8J.zip`
(or newer) from
    https://sourceforge.net/projects/saxon/files/Saxon-HE/9.9/
and place it in the `bin/` folder (next to `console`) and make sure
the path to the `java` binary is properly set in the following
line in `parameters.yaml`:

    app.xslt.adapter.arguments: "/usr/bin/java -jar %kernel.project_dir%/bin/saxon9he.jar -s:%%source%% -xsl:%%xsl%%  %%additional%%"

On Windows, it might look like

    app.xslt.adapter.arguments: "c:\\Run\\Java\\jdk1.8\\bin\\java -jar %kernel.project_dir%\\bin\\saxon9he.jar -s:%%source%% -xsl:%%xsl%% %%additional%%"

depending on your local Java installation.

### Solr Setup
You can skip this installation in the first step. Everything except the
search field should still work.

First, download

    https://archive.apache.org/dist/lucene/solr/6.2.0/solr-6.2.0.zip

and extract the contents of `solr-6.2.0` into the existing `solr/` folder.

Start solr by

    ./solr/bin/solr start

and then create the `demo_de` core

    ./solr/bin/solr create -c demo_de

You can clear the core and re-index existing entities

    ./bin/console solr:index:clear

    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Person"
    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Organization"
    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Place"
    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Bibitem"
    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Event"
    ./bin/console solr:index:populate "TeiEditionBundle\\Entity\\Article"

For trouble-shooting, you can access the Solr admin interface at

    http://localhost:8983/solr/

To stop it again, call

    ./solr/bin/solr stop -all

### Setup Web-Server
For testing purposes, you can use the built-in server from PHP

    php -S localhost:8000 -t web

And then navigate to http://localhost:8000/index.php/

If you are running on a different host than `localhost`, make sure to adjust

    jms_i18n_routing.hosts.de:  localhost

in `config/parameters.yaml` accordingly.

In order to point a proper Web-Server (apache or nginx) to `web`, see
    https://symfony.com/doc/current/setup/web_server_configuration.html for
further detailed instruction.

Make sure to copy `.htaccess.dist` to `.htaccess` if you want to run the site
without prepnding `/index.php/` to every url.

If you get errors due to var not being writable, adjust directory permissions as
described in https://symfony.com/doc/3.4/setup/file_permissions.html
- sudo setfacl -R -m u:www-data:rwX /path/to/var
- sudo setfacl -dR -m u:www-data:rwX /path/to/var

If you get errors due to web/css not being writable, adjust directory permissions as
described in https://symfony.com/doc/3.4/setup/file_permissions.html
- sudo setfacl -R -m u:www-data:rwX /path/to/web/css
- sudo setfacl -dR -m u:www-data:rwX /path/to/web/css

### Troubleshooting

If you experience any issues, please contact us:
    https://jewish-history-online.net/contact

We are looking forward to work this through step by step or provide you
with a preconfigured VirtualBox.

Adding and updating Content
---------------------------
TEI files and page facsimiles are located in `sites/demo-edition/data` in the
corresponding `tei` or `img/source-xxxxx` folders.

Follow the following commands to add the first source to the site:

Add the author (Franz Kafka, by GND 118559230) to the `Person` table:

    ./bin/console article:author --insert-missing sites/demo-site/data/tei/source-00001.de.xml

You should now see it at

    http://localhost:8000/index.php/person

Add the source

    ./bin/console article:header --insert-missing sites/demo-site/data/tei/source-00001.de.xml

Refresh the source (this will fetch every persName / orgName / placeName with GND or TGN identifier)

    ./bin/console article:refresh sites/demo-site/data/tei/source-00001.de.xml

If we have a source with page facsimile as hinted by

    <classCode scheme="http://juedische-geschichte-online.net/doku/#genre">Quelle:Text</classCode>

We can now generate the tiles

    ./bin/console source:tiles sites/demo-site/data/tei/source-00001.de.xml

(`convert` from the ImageMagick packaged is called to generate the tiles in `web/viewer/source-00001/`)

And generate the METS-container needed for `iview`

    ./bin/console source:mets sites/demo-site/data/tei/source-00001.de.xml > web/viewer/source-00001/source-00001.de.mets.xml

You can now preview the source at

    http://localhost:8000/index.php/quelle/source-1

If you make changes, you can update all the metadata by running again

    ./bin/console article:refresh sites/demo-site/data/tei/source-00001.de.xml

If you are happy with the display, you can publish it:

    ./bin/console article:header --publish sites/demo-site/data/tei/source-00001.de.xml

It should now show up on the timeline and connected to the author.

Tweaking the site
-----------------
### Translate messages and routes

    ./bin/console translation:extract de --dir=./src/ --dir=vendor/igdj/tei-edition-bundle --output-dir=./translations --enable-extractor=jms_i18n_routing

Theme-specific translations (TODO: add --intl-icu as soon as https://github.com/schmittjoh/JMSTranslationBundle/pull/551 is merged)

    ./bin/console translation:extract de --dir=./sites/demo-site/templates --output-dir=./sites/demo-site/translations

### Font
We use Noto Serif as Brotschrift (book_typeface)
    https://fonts.google.com/specimen/Noto+Serif

The CSS-Settings are made in `assets/scss/font-definition.scss` as by the
`importPaths` in `scssphp`-settings in `config/config.yaml`.

For PDF-Generation, `app.pdf-generator.arguments` in `config/services.yaml` point
to the font files used.

License
-------
    (C) 2021 Institut für die Geschichte der deutschen Juden,
        Daniel Burckhardt


    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as
    published by the Free Software Foundation, either version 3 of the
    License, or (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
