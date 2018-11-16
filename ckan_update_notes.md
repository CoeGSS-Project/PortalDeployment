# CoeGSS libvirt notes


## 2018-03-16 CKAN

### Backup VM (@services)
```
mkdir ~/CKAN_16_03_2018
cd ~/CKAN_16_03_2018
sudo virsh dumpxml CoeGSS-trusty-REPO >./CoeGSS-trusty-REPO.xml
mcview ./CoeGSS-trusty-REPO.xml

    <disk type='file' device='disk'>                                       
      <driver name='qemu' type='qcow2'/>
      <source file='/home/vmPool/CoeGSS/CoeGSS-trusty-REPO.img'/>
      <backingStore type='file' index='1'>
        <format type='raw'/>
        <source file='/home/vmPool/CoeGSS/../baseVMs/trusty-server-CoeGSS-amd64.img'/>
        <backingStore/>
      </backingStore>
      <target dev='vda' bus='virtio'/>
      <boot order='1'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </disk>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/home/vmPool/CoeGSS/CoeGSS-trusty-REPO-data.img'/>
      <backingStore/>
      <target dev='vdb' bus='virtio'/>
      <alias name='virtio-disk1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x0a' function='0x0'/>
    </disk>


sudo virsh shutdown CoeGSS-trusty-REPO
cp /home/vmPool/CoeGSS/CoeGSS-trusty-REPO.img ./
cp /home/vmPool/CoeGSS/CoeGSS-trusty-REPO-data.img ./

sudo virsh start CoeGSS-trusty-REPO
```

### Backup ckan db (@REPO)

```
mkdir ~/db_backup
cd ~/db_backup
sudo -u postgres pg_dump --format=custom -d ckan_default > ./ckan-2018-03-16.dump
```


### Update CKAN (@REPO)

```
cd ~
###wget http://packaging.ckan.org.s3-eu-west-1.amazonaws.com/python-ckan_2.7-trusty_amd64.deb


sudo /etc/init.d/apache2 stop
sudo su ckan
cd /home/ckan/src/ckan

git config user.email shcherbakov@hlrs.de
git config user.name "CoeGSS CKAN System Account"

git stash
git pull origin master


	From https://github.com/ckan/ckan
	 * branch            master     -> FETCH_HEAD
	Updating 91d3a81..db1d2c8
	error: The following untracked working tree files would be overwritten by merge:
	        ckan/config/supervisor-ckan-worker.conf
	Please move or remove them before you can merge.
	Aborting

mv ckan/config/supervisor-ckan-worker.conf ckan/config/supervisor-ckan-worker.conf.bak
git pull origin master
mcdiff ckan/config/supervisor-ckan-worker.conf ckan/config/supervisor-ckan-worker.conf.bak

git checkout tags/ckan-2.7.3
git stash apply

	warning: Cannot merge binary files: ckan/public/base/images/placeholder-420x220.png (Updated upstream vs. Stashed changes)
	Auto-merging test-core.ini
	CONFLICT (content): Merge conflict in test-core.ini
	Auto-merging setup.py
	CONFLICT (content): Merge conflict in setup.py
	Auto-merging ckanext/reclineview/theme/public/vendor/recline/recline.js
	CONFLICT (content): Merge conflict in ckanext/reclineview/theme/public/vendor/recline/recline.js
	Auto-merging ckan/public/base/images/placeholder-420x220.png
	CONFLICT (content): Merge conflict in ckan/public/base/images/placeholder-420x220.png
	Auto-merging ckan/logic/action/create.py
	Auto-merging ckan/config/solr/schema.xml

git status

	HEAD detached at ckan-2.7.3
	Changes to be committed:
	  (use "git reset HEAD <file>..." to unstage)

	        modified:   ckan/config/solr/schema.xml
	        modified:   ckan/logic/action/create.py
	        modified:   ckan/templates/home/snippets/promoted.html

	Unmerged paths:
	  (use "git reset HEAD <file>..." to unstage)
	  (use "git add <file>..." to mark resolution)

	        both modified:      ckan/public/base/images/placeholder-420x220.png
	        both modified:      ckanext/reclineview/theme/public/vendor/recline/recline.js
	        both modified:      setup.py
	        both modified:      test-core.ini

	Untracked files:
	  (use "git add <file>..." to include in what will be committed)

	        ckan/config/supervisor-ckan-worker.conf.bak
	        ckan/public/base/images/COEGSS_logo_RGB.png
	        ckan/public/base/images/coegss_logo_rgb_small.png
	        ckan/public/base/images/placeholder-420x220.png_bak
	        ckanext/ckanext-ldap/
	        ckanext/extrafields/
	        doc/_latest_release.rst


mcedit setup.py
git add setup.py
mcedit test-core.ini
git add test-core.ini
mcedit ckanext/reclineview/theme/public/vendor/recline/recline.js
	<<<<<<< Updated upstream                                                                                                                                                                                            
	    var bg = new L.TileLayer(mapUrl, {maxZoom: 19, attribution: attribution, subdomains: subdomains});
	=======                                                                                                                                                                                                             
	    /***************************************************************************************************************************************/
	    /*** The following three lines were replaced by the latter due to MapQuest's Direct Tile Access has been discontinued. *****************/
	    /*** Please see: https://github.com/ckan/ckan/pull/3174 for details ********************************************************************/
	    /***************************************************************************************************************************************/
	....
	    //var mapUrl = "//otile{s}-s.mqcdn.com/tiles/1.0.0/osm/{z}/{x}/{y}.png";
	    //var osmAttribution = 'Map data &copy; 2011 OpenStreetMap contributors, Tiles Courtesy of <a href="http://www.mapquest.com/" target="_blank">MapQuest</a> <img src="//developer.mapquest.com/content/osm/mq_log
	    //var bg = new L.TileLayer(mapUrl, {maxZoom: 18, attribution: osmAttribution ,subdomains: '1234'});
	
	    var mapUrl = "//stamen-tiles-{s}.a.ssl.fastly.net/terrain/{z}/{x}/{y}.jpg";
	    var osmAttribution = 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStree
	    var bg = new L.TileLayer(mapUrl, {maxZoom: 18, attribution: osmAttribution});
	                                                                                                                                                                                                                    
	    /***************************************************************************************************************************************/
	>>>>>>> Stashed changes                                                                                                                                                                                             

git add ckanext/reclineview/theme/public/vendor/recline/recline.js
git reset HEAD ckan/public/base/images/placeholder-420x220.png
git checkout stash -- ckan/public/base/images/placeholder-420x220.png

git commit -m WIP
git reset HEAD^

exit
sudo /etc/init.d/apache2 start
sudo /etc/init.d/apache2 status

sudo chown :www-data /home/ckan/src/ckan/ckan/public/base/i18n
###sudo apt remove python-setuptools


# manualy remove all setuptools from /usr/lib/python2.7/dist-packages/ and /usr/local/lib/python2.7/dist-packages/

sudo pip install -r /home/ckan/src/ckan/requirement-setuptools.txt
```

# Last Error

```
SearchError: SOLR returned an error running query: {'sort': 'score desc, metadata_modified desc', 'fq': [u'+capacity:public groups:"topsecret" -dataset_type:showcase -dataset_type:harvest', u'+site_id:"default"', '+state:active'], 'facet.mincount': 1, 'rows': 3, 'facet': u'false', 'facet.limit': '50', 'wt': 'json', 'q': '*:*', 'fl': 'id validated_data_dict'} Error: SolrError(u'Solr responded with an error (HTTP 401): [Reason: HTTP Status 401 -]',)
```

# Rolling back


```
cd ~/CKAN_16_03_2018
sudo virsh shutdown CoeGSS-trusty-REPO
sudo mv /home/vmPool/CoeGSS/CoeGSS-trusty-REPO.img ./CoeGSS-trusty-REPO--failded-update.img
sudo mv /home/vmPool/CoeGSS/CoeGSS-trusty-REPO-data.img ./CoeGSS-trusty-REPO-data--failded-update.img

sudo cp ./CoeGSS-trusty-REPO.img /home/vmPool/CoeGSS/CoeGSS-trusty-REPO.img
sudo cp ./CoeGSS-trusty-REPO-data.img /home/vmPool/CoeGSS/CoeGSS-trusty-REPO-data.img

sudo virsh start CoeGSS-trusty-REPO
```



# Second try

## 2018-03-22 CKAN


```
apache2ctl stop
a2enmod rewrite
```


Create `/etc/apache2/sites-available/ckan_503.conf`:

```
<VirtualHost 0.0.0.0:443>
    ServerName ckan.coegss.hlrs.de
    ServerAlias ckan.coegss.hlrs.de

DocumentRoot "/var/www/html"
# Redirect all request to a 503 return code when in maintenance mode
ErrorDocument 503 /maintenance.html
RewriteEngine on
RewriteCond %{REMOTE_ADDR} !^127\.0\.0\.1$
RewriteCond %{REQUEST_URI} !=/maintenance.html
RewriteRule ^ - [R=503,L]


    SSLEngine on
    SSLCertificateFile /etc/apache2/coegss-certs/coegss.hlrs.de-cert.pem
    SSLCertificateKeyFile /etc/apache2/coegss-certs/coegss.hlrs.de-key.pem
    SSLCertificateChainFile /etc/apache2/coegss-certs/ca.pem
    SSLCACertificateFile /etc/apache2/coegss-certs/ca.pem
</VirtualHost>
```

Create `/var/www/html/maintenance.html`:
```
<!DOCTYPE html>
<html>
    <head>
        <title>Maintenance</title>
        <style>
            body{color:#666;background-color:#f1f1f1;font-family:sans-serif;margin:12%;max-width:50%;}
            h1,h2{color:#333;font-size:4rem;font-weight:400;text-transform:uppercase;}
            h2{color:#333;font-size:2rem;}
            p{font-size:1.5rem;}
        </style>
    </head>
    <body>
        <h1>503</h1>
        <h2>Temporarily Offline</h2>
        <p>This site is currently closed for maintenance. Please check back again soon.</p>
    </body>
</html>
```

```
a2dissite ckan_default.conf
a2ensite ckan_503.conf
service apache2 reload
```


```
sudo su ckan
cd
virtualenv ~/venv
. ~/venv/bin/activate
pip install -U pip
pip install -r /home/ckan/src/ckan/requirement-setuptools.txt
pip install -r /home/ckan/src/ckan/requirements.txt
```

### PIP gave this warning!

```
/home/ckan/venv/local/lib/python2.7/site-packages/pip/_vendor/urllib3/util/ssl_.py:339: SNIMissingWarning: An HTTPS request has been made, but the SNI (Subject Name Indication) extension to TLS is not available o
n this platform. This may cause the server to present an incorrect TLS certificate, which can cause validation failures. You can upgrade to a newer version of Python to solve this. For more information, see https
://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  SNIMissingWarning
/home/ckan/venv/local/lib/python2.7/site-packages/pip/_vendor/urllib3/util/ssl_.py:137: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately
 and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecurePlatformWarning

```


```
pip install celery==3.1.24

sudo mcedit  /etc/supervisor/conf.d/supervisor-ckan-worker.conf

	;command=paster --plugin=ckan celeryd --config=/etc/ckan/default/production.ini
	command=/home/ckan/venv/bin/paster --plugin=ckan celeryd --config=/etc/ckan/default/production.ini

sudo /etc/init.d/supervisor reload
#sudo supervisorctl restart all


sudo a2dissite ckan_503.conf
sudo a2ensite ckan_default.conf
sudo service apache2 reload
```


```
sudo apt-get update
sudo apt-get dist-upgrade
```



```

mcedit /etc/apache2/sites-available/ckan_default.conf

    # Deploy as a daemon (avoids conflicts between CKAN instances).
#    WSGIDaemonProcess ckan_default display-name=ckan_default processes=2 threads=15
    WSGIDaemonProcess ckan_default display-name=ckan_default processes=2 threads=15 python-home=/home/ckan/venv


sudo service apache2 reload
```


```
sudo su ckan
. ~/venv/bin/activate
cd ~/src/ckanext-disqus/
python ./setup.py install
cd ~/src/ckanext-coegss_theme/
chown -R ckan:ckan ~/src/ckanext-coegss_theme
python ./setup.py install
cd ~/src/ckanext-extractor/
sudo chown -R ckan:ckan ~/src/ckanext-extractor
python ./setup.py install
sudo chown -R ckan:ckan ~/src/ckanext-harvest/
cd ~/src/ckanext-harvest/   #f4af291754276a0abda0f76bfbf1a5817b2edf7a
python ./setup.py install
sudo chown -R ckan:ckan ~/src/ckanext-shibboleth/
cd ~/src/ckanext-shibboleth/
python ./setup.py install

pip install ckanext-showcase ckanapi  ckanserviceprovider python-ldap
#ckanext-drel

cd ~/src
cp -r /home/burak/ckanext-drel/ ./
cd ~/src/ckanext-drel
python ./setup.py install
sudo chown -R ckan:ckan ~/src/ckan/ckanext/ckanext-ldap
cd ~/src/ckan/ckanext/ckanext-ldap


cd ~/src/ckanext-harvest/
git pull
python ./setup.py install
pip install -r ./pip-requirements.txt 
pip install -e .



cd ~/src/ckan/
pip install -e .
cd ~/src/ckanext-extractor/
pip install -e .
cd ~/src/ckan/ckanext/ckanext-ldap
pip install -e .
cd ~/src/ckanext-disqus/
pip install -e .


```


[Starting from v2.6.0 2016-11-02](http://docs.ckan.org/en/ckan-2.7.3/changelog.html#v2-6-0-2016-11-02) ckan uses `pysolr` instead of `solrpy`

[pysolr doesn't support HTTP Basic Auth](https://github.com/django-haystack/pysolr/pull/65)

And it is not yet released to CKAN - [see this PR](https://github.com/ckan/ckan/pull/4090)


## Pulling latest ckan (dev)

```
sudo su ckan
cd ~/src/ckan
git stash
git pull
git checkout master
	Previous HEAD position was bc4f8df... Fix migration script for #4042
git pull
git stash pop

	Auto-merging setup.py
	Auto-merging ckan/templates/home/snippets/promoted.html
	CONFLICT (content): Merge conflict in ckan/templates/home/snippets/promoted.html
	Auto-merging ckan/logic/action/create.py

mcedit ckan/templates/home/snippets/promoted.html
git commit -m WIP
git reset HEAD^

exit

sudo service apache2 reload
sudo supervisorctl restart all


sudo su ckan
. ~/venv/bin/activate
pip install --upgrade -r requirements.txt
python setup.py develop
/home/ckan/venv/bin/paster --plugin=ckan search-index rebuild -r --config=/etc/ckan/default/production.ini

	ckan.lib.search.common.SearchError: SOLR schema version not supported: 2.3. Supported versions are [2.8]

sudo mv /opt/solr/example/solr/core0/conf/schema.xml /opt/solr/example/solr/core0/conf/schema.xml.bak
sudo ln -s /home/ckan/src/ckan/ckan/config/solr/schema.xml /opt/solr/example/solr/core0/conf/schema.xml
sudo service tomcat7 restart

/home/ckan/venv/bin/paster db upgrade -c /etc/ckan/default/production.ini
/home/ckan/venv/bin/paster search-index rebuild -r --config=/etc/ckan/default/development.ini

```


# 26.3.2018

Checkout to `ckan-2.7.3`
Applied [this PR](https://github.com/ckan/ckan/pull/4090) as patch.



<s>
```
#sudo -u postgres dropdb ckan_default
#sudo -u postgres pg_restore --format=custom -C -d ckan_default ./ckan-2018-03-22.dump
```
</s>

```
sudo -u postgres pg_restore -c -d ckan_default ./ckan-2018-03-22.dump
sudo -u www-data /home/ckan/venv/bin/paster db upgrade -c /etc/ckan/default/production.ini
```


## Content of `/home/ckan/src/ckan/ckan/lib/search/common.py`

```
# encoding: utf-8

import datetime
import logging
import re
import pysolr
import simplejson
import urllib
log = logging.getLogger(__name__)


class SearchIndexError(Exception):
    pass


class SearchError(Exception):
    pass


class SearchQueryError(SearchError):
    pass


DEFAULT_SOLR_URL = 'http://127.0.0.1:8983/solr'


class SolrSettings(object):
    _is_initialised = False
    _url = None
    _user = None
    _password = None

    @classmethod
    def init(cls, url, user=None, password=None):
        if url is not None:
            cls._url = url
            cls._user = user
            cls._password = password
        else:
            cls._url = DEFAULT_SOLR_URL
        cls._is_initialised = True

    @classmethod
    def get(cls):
        if not cls._is_initialised:
            raise SearchIndexError('SOLR URL not initialised')
        if not cls._url:
            raise SearchIndexError('SOLR URL is blank')
        return (cls._url, cls._user, cls._password)


def is_available():
    """
    Return true if we can successfully connect to Solr.
    """
    try:
        conn = make_connection()
        conn.search(q="*:*", rows=1)
    except Exception, e:
        log.exception(e)
        return False
    return True


def make_connection(decode_dates=True):
    solr_url, solr_user, solr_password = SolrSettings.get()

    if solr_url and solr_user:
        # Rebuild the URL with the username/password
        protocol = re.search('http(?:s)?://', solr_url).group()
        solr_url = re.sub(protocol, '', solr_url)
        solr_url = "{}{}:{}@{}".format(protocol,
                                       urllib.quote_plus(solr_user),
                                       urllib.quote_plus(solr_password),
                                       solr_url)

    if decode_dates:
        decoder = simplejson.JSONDecoder(object_hook=solr_datetime_decoder)
        return pysolr.Solr(solr_url, decoder=decoder)
    else:
        return pysolr.Solr(solr_url)


def solr_datetime_decoder(d):
    for k, v in d.items():
        if isinstance(v, basestring):
            possible_datetime = re.search(pysolr.DATETIME_REGEX, v)
            if possible_datetime:
                date_values = possible_datetime.groupdict()
                for dk, dv in date_values.items():
                    date_values[dk] = int(dv)

                d[k] = datetime.datetime(date_values['year'],
                                         date_values['month'],
                                         date_values['day'],
                                         date_values['hour'],
                                         date_values['minute'],
                                         date_values['second'])
    return d
```









<!---

```
git stash
git checkout ckan-2.7.3
git stash pop
	Auto-merging setup.py
	Auto-merging ckan/templates/home/snippets/promoted.html
	CONFLICT (content): Merge conflict in ckan/templates/home/snippets/promoted.html
	Auto-merging ckan/logic/action/create.py
	Auto-merging ckan/lib/search/common.py
	CONFLICT (content): Merge conflict in ckan/lib/search/common.py

mcview ckan/lib/search/common.py
mcview ckan/templates/home/snippets/promoted.html

git reset HEAD ckan/templates/home/snippets/promoted.html
git checkout stash -- ckan/templates/home/snippets/promoted.html

git reset HEAD ckan/lib/search/common.py
git checkout stash@{1} -- ckan/lib/search/common.py  #stash before going to latest (master) release

git commit -m WIP
git reset HEAD^

### edit like here - https://github.com/ckan/ckan/pull/4090
mcedit ckan/lib/search/common.py

	import urllib

	    if solr_url and solr_user:
        # Rebuild the URL with the username/password
        protocol = re.search('http(?:s)?://', solr_url).group()
        solr_url = re.sub(protocol, '', solr_url)
        solr_url = "{}{}:{}@{}".format(protocol,
                                       urllib.quote_plus(solr_user),
                                       urllib.quote_plus(solr_password),
                                       solr_url)


a2dissite ckan_503.conf
a2ensite ckan_default.conf
sudo service apache2 reload
sudo supervisorctl restart all

```
--->