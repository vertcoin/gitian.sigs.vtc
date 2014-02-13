gitian.sigs.vtc
===============

This repository holds the build signatures for the gitian build process.

Gitian-builder (https://github.com/devrandom/gitian-builder) enables deterministic, replicable build inside a virtual machine,
such that several people can build from the same source and generate a bit-for-bit identical set of binaries. This enhances
the trustlessness of the Vertcoin build/release process, by allowing anyone to validate that the released binaries were
generated from the unchanged published source code, even if they don't build the binaries themselves - a user can assure
themselves that multiple individuals claimed to have generated the same results from the same source code, eliminating
the possibility of a single individual concealing changes to the code.

The more people that gitian build, the stronger the assurance. Please consider contacting contact@vertcoin.org if you
are able to gitian build, so you can be added to the repository as a contributor and push up your build assert files
and signatures.

HOWTO for Gitian builds:
========================

Starting with a clean install of Ubuntu 12.04 LTS x64, this HOWTO was written/tested against Vertcoin 0.8.6.2. If you are 
building a later version, you may need to make small adjustments based on updated versions of dependencies etc.

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python-vm-builder qemu-kvm apt-cacher ruby git

If asked for a mode for apt-cacher to run in, select daemon

To start KVM, sudo modprobe kvm_intel or kvm_amd depending on your architecture, and sudo modprobe kvm in either case, 
and add them to /etc/modules if you want it loaded on boot in future. 

NOTE - if you're doing this in a VM, you'll need to have the functionality to pass through virtualization extensions to the VM, 
so you can nested-virtualise. In recent VMWare workstation versions, this is available as the "Virtualize Intel VT-x/EPT or AMD-V/RVI" 
setting in the Processors section of the VM settings, availability of this functionality and what it's called if available will 
obviously vary. If you are doing this in a VM and you problems starting the nested VMs to make the builds, it could be that you're out 
of memory - check the var/ folder in gitian-builder for logs and allocate enough memory - 4GB should do, you might get away with less.

git clone https://github.com/devrandom/gitian-builder.git

git clone https://github.com/vertcoin/gitian.sigs.vtc

git clone https://github.com/vertcoin/vertcoin

Now make sure you're working on the correct version of the source that was used to build the release you are verifying. 

cd vertcoin
git checkout v0.8.6.2

Now create a git branch of the gitian.sigs.vtc repository, so you can generate a pull request later once you've signed your 
build assert files:

cd ../gitian-builder
git checkout -b mysigs

Now go and create a fork of the gitian.sigs.vtc repository on github, and add it as another remote with:

git remote add myfork url_of_your_new_fork_repo

and build the base VM in which the builds take place:

sudo bin/make-base-vm --arch amd64 --suite precise

Now we need to get some of the dependencies for building Vertcoin, so, still in gitian-builder:

mkdir inputs
cd inputs

wget 'http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.6.tar.gz' -O miniupnpc-1.6.tar.gz
wget 'http://www.openssl.org/source/openssl-1.0.1c.tar.gz'; wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz' 
wget 'http://downloads.sourceforge.net/project/libpng/zlib/1.2.6/zlib-1.2.6.tar.gz'
wget 'http://sourceforge.net/projects/libpng/files/libpng15/older-releases/1.5.9/libpng-1.5.9.tar.gz' 
wget 'http://fukuchi.org/works/qrencode/qrencode-3.2.0.tar.bz2'
wget 'http://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2' 
wget 'https://svn.boost.org/trac/boost/raw-attachment/ticket/7262/boost-mingw.patch' -O boost-mingw-gas-cross-compile-2013-03-03.patch 
wget 'http://download.qt-project.org/archive/qt/4.8/4.8.3/qt-everywhere-opensource-src-4.8.3.tar.gz'

Now we can start to gitian-build the dependencies which will be needed to actually build Vertcoin. They must be built in gitian too, 
so we have a replicable set of dependencies - this will ensure the final binaries are also replicable/deterministic. 

sudo bin/gbuild ../vertcoin/contrib/gitian-descriptors/boost-win32.yml

Expect this to take a while - the boost build is a big one.

At this point, when building on a completely clean, unmodified Ubuntu 12.04 x64 desktop edition, I sometimes get a popup message 
saying a system error has occured. If this happens - just cancel it, it doesn't seem to affect my builds, and I am not certain what 
exactly the problem is (if any).

The build will run, and will export the results to the build/out folder in gitian-builder. 

Each time you build a dependency, you must move it to the "inputs" folder, since the build/out folder is wiped during each gitian-build, 
and we need all these dependencies in the inputs folder ready for the final build.

sudo mv build/out/boost-win32-1.55.0-gitian-r6.zip inputs/

sudo ./bin/gbuild ../vertcoin/contrib/gitian-descriptors/deps-win32.yml

sudo mv build/out/bitcoin-deps-win32-gitian-r9.zip inputs/

sudo ./bin/gbuild ../vertcoin/contrib/gitian-descriptors/qt-win32.yml

sudo mv build/out/qt-win32-4.8.3-gitian-r4.zip inputs/

Now we can build Vertcoin, note that you must specify the tag in the repository that relates to the version you wish to build:

sudo bin/gbuild --commit vertcoin=v0.8.6.2 ../vertcoin/contrib/gitian-descriptors/gitian-win32.yml

It will pull the source again from git, into the inputs folder. The reason we have specified the version twice (earlier when we 
did 'git checkout', and again now), is because earlier we wanted to ensure we were working with the correct version of the gitian 
descriptors, now we are telling it what commit we actually want to build. 

You will now see gitian pull in all the dependencies and run the build inside the VM.

When it's done, it will again generate output in the build/out folder, this time, the Vertcoin binaries. 	

You also now have a vertcoin-res.yml file in the result/ folder, asserting the SHA256 hashes of all the files involved in the build, 
and at the end, the resultant binaries.

You can verify the signatures of the signers who have already gitian build, by cd'ing into the appropriate subdirectory of 
gitian.sigs.vtc and doing:

gpg --verify vertcoin-build.assert.sig vertcoin-build.assert

If you don't have the public keys of the other signers on your keyring yet, then cd into the pubkeys folder and do:

gpg --import *.asc

If you're unfamiliar with GPG, http://www.gnupg.org/gph/en/manual.html is a very good getting started guide.

To validate the builds as a user, all you need to is check that the signatures are good, then diff the vertcoin-build.assert files 
of several signers to ensure they are all identical, thereby assuring you that the files were generated by the named individuals, 
and that they each built the same source and generated the same resultant binaries. 

To get the SHA256 hashes of the binaries (to compare to the build assert files), http://sourceforge.net/projects/quickhash/ 
(among others) works well.

Now you can get to signing your assert files:

- If you don't have your gpg key on the build VM yet, simply gpg --import it first, then;

bin/gsign --signer you@yourdomain.com --release v.0.8.6.2 --destination ../gitian.sigs.vtc/ ../vertcoin/contrib/gitian-descriptors/gitian-win32.yml

You now have an .assert file in the appropriate subfolder of the gitian.sigs.vtc folder, and an accompanying signature.

Don't forget, if this is the first time you've gitian built, to also:

cd ../gitian.sigs.vtc/pubkeys

gpg --export -a "User Name" > "Your Name you@yourdomain.com (0xKeyFingerprint) pub.asc

Now simply commit your signature back to the github.

cd ../gitian.sigs.vtc/
git add -A
git commit -a
git push myfork mysigs

Now you can either manually git request-pull from the command line, or go to your forked repo on the github website, and
click the button to create a pull request. If you do it from the command line, remember to email the output to contact@vertcoin.org
however doing it from the github site automatically does this.

