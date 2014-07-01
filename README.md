# Pelagos

Pelagos global fishing vessel visualization


# Installation

Install (unpack) the Google AppEngine SDK somewhere. The following documentation assumes you have installed it in `~/Projects/skytruth/google_appengine`. You will need to adjust the commands accordingly if you chose a different location.

    git submodule update --init --recursive
    virtualenv deps
    source deps/bin/activate
    pip install -r requirements.txt


## GAE Virtualenv Setup

We have a special module that loads a virtualenv at runtime in app engine. This is useful because then we can pip install any dependencies that need to be deployed along with app to GAE. Before you deploy, do the following to initialize the virtual env (from the project root)

    virtualenv gae/virtualenvloader/gaevirtualenv
    source gae/virtualenvloader/gaevirtualenv/bin/activate
    pip install -r gae-requirements.txt

# Setup

    source deps/bin/activate
    export PATH=$PATH:~/Projects/skytruth/google_appengine/
    export PYTHONPATH=$PYTHONPATH:~/Projects/skytruth/google_appengine

# Testing

To test the app engine code, run everything from the gae folder

    cd gae

Run unit tests in gae/tests with nose

    nosetests --with-gae --without-sandbox

You may also want to add this to reduce output if a test fails

    nosetests --with-gae --without-sandbox --logging-level=WARNING

And if your GAE SDK is not installed in the default location then you will also need something like

    nosetests --with-gae --without-sandbox --logging-level=WARNING --gae-lib-root=~/Projects/skytruth/google_appengine

Start up a local dev server.  To do this you will need to provide [authentication credentials](/auth.md) for a service account that has access to a BigQuery project.  See explanation in the next section.

    $ dev_appserver.py --appidentity_email_address xxxx@developer.gserviceaccount.com --appidentity_private_key_path /path/to/secret.pem .

Run client-side (in browser) unit tests in gae/tests/client-tests

    http://localhost:8080/test/index.html


## BigQuery

Unfortunately there is no mechanism for unittesting with BigQuery in App Engine, so we built a little [bigquery wrapper class](/gae/bigquery.py) to allow for nosetests to exercise code that depends on it.

You can load a [fixture](/gae/test/fixtures) into memory and have bigquery results served from that im-memory result set instead of interacting with the BigQuery API.

The fixture mechanism works by matching the sql query generated by the tilegenerator with a stored result in the fixture. If the sql query matches exactly, the stores result set is returned.

To re-generate the fixtures (for example if you change the structure of the sql), run this:

    curl http://localhost:8080/admin/bqfixture -o tests/fixtures/bigquery.json

# API

The tile server has a simple restful API that returns a tiles containing vector data packed in TypedMatrix format

The _tile_ api will return a single tile with each request

    Url format: /tile/<tile_set>/<bbox>

## BBox (Bounding Box) Format
A BBox is provided as a comma separated list
    lonmin, latmin, lonmax, latmax


## Test Tilesets
Test tilesets are provided
    /tile/test-static-bin/<bbox>
will always return the same tile data. The bbox parameter is ignored.  The data comes from the static file [a relative link](gae/data/sample_tile.bin)
    /tile/test-static-json/<bbox>
will always return the same tile data. The bbox parameter is ignored.  The data comes from the static file [a relative link](gae/data/sample_tile.json)
    /tile/test-random/<bbox>
will return 1000 random points inside the given bbox

