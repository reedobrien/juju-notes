# Writing a CI test for a juju core feature

## System info

	$ uname -a
		Linux erwin 4.4.0-16-generic #32-Ubuntu SMP Thu Mar 24 22:38:01 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
	$ lsb_release -a
		No LSB modules are available.
		Distributor ID:	Ubuntu
		Description:	Ubuntu Xenial Xerus (development branch)
		Release:	16.04
		Codename:	xenial 
	$ go version
	    go version go1.6 linux/amd64 

	$ sudo apt-get install lxd
		...

## Build Juju

	$ mkdir ~/juju
	$ gopath ~/juju
	$ export PATH=$PATH:$HOME/juju/bin
	$ mkdir juju/src/github.com/juju/juju
	$ cd  juju/src/github.com/juju/juju
	$ git clone git@github.com:juju/juju.git .
	$ go get ./...
	$ godeps -u depedencies.txt
	$ make
	$ go install ./...
	$ go test ./... TODO(ro) BTA duration too long. Only features that take longer than a test run become targets.


## Try it out

	$ export CONTROLLER_NAME=lxd-controller
	$ juju bootstrap $CONTROLLER_NAME lxd --upload-tools
	$ juju deploy mysql && juju deploy wordpress
	$ juju add-relation wordpress mysql
	$ juju expose wordpress
	$ juju status  ## note the wordpress IP
	$ xdg-open http://<wordpress_address>

Fill in the form and setup a blog... good enough if it worked. Destry it NOW,
before you update master and build a new juju, the new version cannot -- as of
this writing -- destroy a controller that was created by a preveious version. 

	$ juju destroy-controller $CONTROLLER_NAME


## Get ci tools and ci repository 

	$ cd ~/py
	$ bzr branch lp:juju-ci-tools/repository
	$ bzr branch lp:juju-ci-tools 
	$ cd juju-ci-tools

Other strange magic to use the CI test tools. TODO(ro) BTA WTF to get it setup.

	$ mkvirtualenv juju-ci-tools
	$ pip install -r requirements.txt
	$ pip install python-yaml
	$ pip install py-yaml
	$ pip install pyyaml
	$ pip install boto
	$ pip install mock
	$ pip install python-jenkins
	$ pip install python-novaclient 
	$ mkdir ~/.juju  # Sadly the CI Tools require this for 2.x tests too. Martin is aware, but ...
	$ vim ~/.juju/environments.yaml  
		environments:
			lxd:
				type: lxd
				# this should make the lxd provider skip apt-get up{date|grade}
				# which is slow, particularly on the hotel network. Alas, it
				# doesn't work on my system.
				enable-os-upgrades: false 

	$ source ~/.aws/ro  ## Load some aws creds in the environment
	$ juju autoload-credentials  # This pulls the aws creds and write it to a creds file
	$ ln -s ~/.local/share/juju/credentials.yaml ~/.juju/  ## Link the creds file to the juju1 location for ci-tools
	$ touch ~/.juju/clouds.yaml  # ci-tools requires this file, even if empty.
	$ mkdir ~/tmp/juju-ci-tools  # ci-tools puts output here.

Next we need to make a test suite for ci-tools 

	$ make new-assess
		install -m 755 template_assess.py.tmpl assess_NAMEHERE.py
		install -m 644 template_test.py.tmpl tests/test_assess_NAMEHERE.py
		sed -i -e "s/TEMPLATE/NAMEHERE/g" assess_NAMEHERE.py tests/test_assess_NAMEHERE.
	$ # Rename the files with your fetaure branch in lieu of NAMEHERE
	$ # Search/replace internally to replace NAMEHERE with your feature moniker. 
	$ # Edit edit edit
	$ rm -rf ~/tmp/juju-ci-tools/*  # Empty this NOW, so it doesn't bomb after a run and you don't get your logs you might want.
	$ export TEST_CONTROLLER_NAME=test-lxd-controller-name
	$ date && python assess_min_version.py --series xenial --debug lxd ~/juju/bin/juju ~/tmp/juju-ci-tools --upload-tools  && echo "exit: $?"; date 	  $ Wait...



## Barriers to adoption

...
