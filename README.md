Sobre as ferramentas YETIsense
========================

Em conjunto com src.git, ports.git, core.git e plugins.git, eles
criam conjuntos, pacotes e imagens para o projeto YETIsense.

Configurando um sistema de construção
=========================

Instale o [FreeBSD](https://www.freebsd.org/) 14.2-RELEASE para amd64
em uma máquina com pelo menos 50 GB de disco rígido, pelo menos 18 GB de RAM e 16vCPU 
para construir com sucesso todas as imagens padrão. Todas as tarefas exigem um
usuário root. Faça o seguinte para obter os repositórios (sobrescrevendo portas e src padrão):

    # pkg install -y git portsnap
    # mkdir /var/db/portsnap
    # portsnap fetch extract
    # make clean
    # make -C /usr/ports/ports-mgmt/pkg clean all reinstall
    # cd /usr
    # git clone https://github.com/Christopher-YeTI/tools
    # cd tools
    # chmod +x /usr/tools/scripts/pkg_fingerprint.sh
    # make update

Note que os repositórios YETIsense também podem ser configurados em um diretório não-/usr
configurando ROOTDIR. Por exemplo:

    # mkdir -p /tmp/yetisense
    # cd /tmp/yetisense
    # git clone https://github.com/Christopher-YeTI/tools
    # cd tools
    # env ROOTDIR=/tmp/yetisense make update

TL;DR
=====

    # make dvd

Se for bem-sucedido, uma imagem de DVD pode ser encontrada em:

    # /usr/local/yetisense/build/25.1/amd64/images/xxx.iso

    # make print-IMAGESDIR

Etapas e opções detalhadas de construção
================================

Como especificar opções de compilação na linha de comando
------------------------------------------------

A construção é dividida em estágios individuais: base,
kernel, ports, plugins e core podem ser construídos separadamente e
repetidamente sem afetar os outros estágios. Todos os estágios
podem ser reinvocados e continuar a construção sem limpar o
progresso anterior. Um estágio final reúne todos os cinco estágios
em uma imagem de destino.

Todos os passos de construção são invocados via make(1):

    # make step OPTION="value"

As opções de construção inicial disponíveis são:

* SETTINGS: o nome da configuração local solicitada
* CONFIGDIR: leia a configuração de outro diretório e substitua SETTINGS
(certifique-se de usar um caminho absoluto ao especificar)

As opções de compilação disponíveis são:

* ABI: uma ABI personalizada (padrão para SETTINGS)
* ADDITIONS: uma lista de pacotes/plugins para adicionar às imagens
* ARCH: a arquitetura de destino se não for nativa
* COMSPEED: velocidade serial, por exemplo, "115200" (padrão)
* DEBUG: constrói um kernel de depuração com informações adicionais do objeto
* DEVICE: carrega modificações específicas do dispositivo, por exemplo, "A10" (padrão)
* KERNEL: a configuração do kernel a ser usada, por exemplo, SMP (padrão)
* MIRRORS: uma lista de espelhos para pré-buscar conjuntos
* NAME: "YETIsense" (padrão)
* PRIVKEY: a chave privada para assinar conjuntos
* PUBKEY: a chave pública para assinar conjuntos
* SUFFIX: o sufixo do nome do pacote superior (o padrão é vazio)
* TYPE: o nome base do pacote superior a ser instalado
* UEFI: use imagens híbridas amd64 para essas imagens, por exemplo, "vga vm"
* VERSION: uma tag de versão (se aplicável)
* ZFS: nome do pool ZFS a ser criado para imagens de VM, por exemplo, "zpool"

Como especificar opções de construção por meio de arquivo de configuração
---------------------------------------------------

O arquivo de configuração é necessário em "CONFIGDIR/build.conf".
Seu conteúdo pode ser modificado para adaptar um ambiente de construção não padrão
e para evitar argumentos Makefile excessivos.

Uma substituição local existe como "CONFIGDIR/build.conf.local" e é
analisada primeiro para permitir substituições mais flexíveis. Use com cuidado.

Como executar etapas de construção individuais ou compostas
----------------------------------------------

Kernel, base, packages and release sets are stored under:

    # make print-SETSDIR

All final images are stored under:

    # make print-IMAGESDIR

Build the userland binaries, bootloader and administrative files:

    # make base

Build the kernel and loadable kernel modules:

    # make kernel

Build all the third-party ports:

    # make ports

Build additional plugins if needed:

    # make plugins

Wrap up our core as a package:

    # make core

A dvd live image is created using:

    # make dvd

A serial memstick live image is created using:

    # make serial

A vga memstick live image is created using:

    # make vga

A flash card full disk image is created using:

    # make nano

A virtual machine full disk image is created using:

    # make vm

A special embedded device image based on vm variety:

    # make factory

Release sets can be built as follows although the result is
an unpredictable set of images depending on the previous
build states:

    # make release

However, the release target is necessary for the following
target which includes sanity checks, proper clearing of the
images directory and core package version alignment:

    # make distribution

Cross-building for other architecures
-------------------------------------

This feature is currently experimental and requires installation
of packages for cross building / user mode emulation and additional
boot files to be installed as prompted by the build system.

A cross-build on the operating system sources is executed by
specifying the target architecture and custom kernel:

    # make base kernel DEVICE=BANANAPI

In order to speed up building of using an emulated packages build,
the xtools set can be created like so:

    # make xtools DEVICE=BANANAPI

The xtools set is then used during the packages build similar to
the distfiles set.

    # make packages DEVICE=BANANAPI

The final image is built using:

    # make arm-<size> DEVICE=BANANAPI

Currently available device are: BANANAPI and RPI2.

About other scripts and tweaks
==============================

Device-specific settings
------------------------

Device-specific settings can be found and added in the
device/ directory.  Of special interest are hooks into
the build process for required non-default settings for
image builds.  The .conf files are shell scripts that can
define hooks in the form of e.g.:

    serial_hook()
    {
        # ${1} is the target file system root
        touch ${1}/my_custom_file
    }

These hooks are available for all image types, namely
dvd, nano, serial, vga and vm.  Device-specific hooks
are loaded after config-specific hooks and both of them
can coexist in a given build.

Updating the code repositories
------------------------------

Updating all or individual repositories can be done as follows:

    # make update[-<repo1>[,...]] [VERSION=git.tag]

Available update options are: core, plugins, ports, portsref, src, tools

VERSION can be used to update to the matching git tag instead of HEAD.

Regression tests and ports audit
--------------------------------

Before building images, you can run the regression tests
to check the integrity of your core.git modifications plus
generate output for the style checker:

    # make test

To check the binary packages from ports against the upstream
vulnerability database run the following:

    # make audit

Advanced package builds
-----------------------

Package sets ready for web server deployment are automatically
generated and modified by ports, plugins and core steps.  The
build automatically caches temporary build dependencies to avoid
spurious rebuilds.  These packages are later discarded to provide
a slim runtime set only.

If signing keys are available, the packages set will be signed
twice, first embedded into repository metadata (inside) and
then again as a flat file (outside) to ensure integrity.

For faster ports building it may be of use to cache all distribution
files before running the actual build:

    # make distfiles

For targeted rebuilding of already built packages the following
works:

    # make ports-<packagename>[,...]
    # make plugins-<packagename>[,...]
    # make core-<packagename>[,...]

Please note that reissuing ports builds will clear plugins and
core progress.  However, following option apply to PORTSENV:

* BATCH=no	Developer mode with shell after each build failure
* DEPEND=no	Do not tamper with plugins or core packages
* MISMATCH=no	Rebuild packages that have a version mismatch
* PRUNE=no	Do not check ports integrity prior to rebuild

The defaults for these ports options are set to "yes".  A sample
invoke is as follows:

    # make ports-curl PORTSENV="DEPEND=no PRUNE=no"

Both ports and plugins builds allow to override the current list
derived from their respective configuration files, i.e.:

    # make ports PORTSLIST="security/openssl"
    # make plugins PLUGINSLIST="devel/debug"

Acquiring precompiled sets from the mirrors or another local directory
---------------------------------------------------------------------

Compiled sets can be prefetched from a mirror if they exist,
while removing any previously available set:

    # make prefetch-<option>[,...] [VERSION=<full_version>]

If another build configuration is used locally that is compatible,
the sets can be cloned from there as well:

    # make clone-<option>[,...] TO=<major_version>

Available prefetch or clone options are:

* base:		select matching base set
* distfiles:	select matching distfiles set (clone only)
* kernel:	select matching kernel set
* packages:	select matching packages set

Using signatures to verify integrity
------------------------------------

Signing for all sets can be redone or applied to a previous run
that did not sign by invoking:

    # make sign-base,kernel,packages

A verification of all available set signatures is done via:

    # make verify

Nano image size adjustment
--------------------------

Nano images can be adjusted in size using an argument as follows:

    # make nano-<size>

Virtual machine images
----------------------

Virtual machine images come in varying disk formats and sizes.
For this reason they are not included in our binary releases.
The default format is vmdk with 20G and 1G swap.  If you want
to change that you can manually alter the invoke using:

    # make vm-<format>[,<size>[,<swap>[,<extras>]]]

Available virtual machine disk formats are:

* qcow:		Qemu, KVM (legacy format)
* qcow2:	Qemu, KVM (not backwards-compatible)
* raw:		Unformatted (sector by sector)
* vhd:		VirtualPC, Hyper-V, Xen (dynamic size)
* vhdf:		Azure, VirtualPC, Hyper-V, Xen (fixed size)
* vmdk:		VMWare, VirtualBox (dynamic size)

The swap argument is either its size or set to "off" to disable.

The extras argument can be any extras.conf hook in case the
default "vm" hook is not desirable.

Clearing individual build step progress
---------------------------------------

A couple of build machine cleanup helpers are available
via the clean script:

    # make clean-<option>[,...]

Available clean options are:

* arm:		remove arm image
* base:		remove base set
* distfiles:	remove distfiles set
* dvd:		remove dvd image
* core:		remove core from packages set
* images:	remove all images
* kernel:	remove kernel set
* logs:		remove all logs
* nano:		remove nano image
* obj:		remove all object directories
* packages:	remove packages set
* plugins:	remove plugins from packages set
* ports:	alias for "packages" option
* release:	remove release set
* serial:	remove serial image
* sets:		remove all sets
* src:		reset kernel/base build directory
* stage:	reset main staging area
* vga:		remove vga image
* vm:		remove vm image
* xtools:	remove xtools set

How the port tree is updated via its upstream repository
--------------------------------------------------------

The ports tree has a few of our modifications and is sometimes a
bit ahead of FreeBSD.  In order to keep the local changes, a
skimming script is used to review and copy upstream changes:

    # make skim[-<option>]

Available options are:

* used:		review and copy upstream changes
* unused:	copy unused upstream changes
* (none):	all of the above

Syncing a ports branch for custom package builds
------------------------------------------------

When maintaining branches the master branch holds updates that
we want to cherry-pick to another branch.  To ease the process
the sync step can deal with the complexity involved:

    # make sync-category/port[,category/port[,...]]

Rebasing the file lists for the base sets
-----------------------------------------

In case base files changed, the base package list and obsoleted
files need to be regenerated.  This is done using:

    # make rebase

Switching to the build jail for inspection
------------------------------------------

Shall any debugging be needed inside the build jail, the following
command will use chroot(8) to enter the active build jail:

    # make chroot[-<subdir>]

Boot images in the native bhyve(8) hypervisor
---------------------------------------------

There's also the posh way to boot a final image using bhyve(8):

    # make boot-<image>

Please note that login is only possible via the Nano and Serial images.

Booting VM images will not work for types other than "raw".

Generating a make.conf for use in running YETIsense
--------------------------------------------------

A ports tree in a running YETIsense can be used to build packages
not published on the mirrors.  To generate the make.conf contents
for standalone use on the host use:

    # make make.conf

Reading and modifying version numbers of build sets and images
--------------------------------------------------------------

Normally the build scripts will pick up version numbers based
on commit tags or given version tags or a date-type string.
Should it not fit your needs, you can change the name using:

    # make rename-<set>[,<another_set>] VERSION=<new_name>

The available targets are: base, distfiles, dvd, kernel, nano,
packages, serial, vga and vm.

The current state of the associated build repositories checked
out on the system can be printed using:

    # make info

Repositories that have signing keys can show the current
fingerprint using:

    # make fingerprint

Last but not least, in case build variables needs to be inspected,
they can be printed selectively using:

    # make print-<variable1>[,<variable2>]

Compressing images
------------------

Images are compressed using bzip2(1) for distribution.  This can
be invoked manually using:

    # make compress-<image1>[,<image2>]

Composite build steps
---------------------

A fully contained nightly build for the system is invoked using:

    # make nightly

When nightly builds are being run you can get a brief report of
the latest one for each build step or select a build step to either
view the file or watch it run in real time:

    # make watch[-<step>]

To allow the nightly build to build both release and development packages
use:

    # make nightly EXTRABRANCH=master

Nightly builds are the only builds that write and archive logs under:

    # make print-LOGSDIR

with ./latest containing the last nightly build run.  Older logs are
archived and available for a whole week for retrospective analysis.

To push sets and images to a remote location use the upload target:

    # make upload-<set>[,...]

To pull sets and images from a remote location use the download target:

    # make download-<set>[,...]

Logs can be downloaded as well for local inspection.  Note that download
like prefetch will purge all locally existing targets.  Use SERVER to
specify the remote end, e.g. SERVER=user@does.not.exist

Additionally, UPLOADDIR can be used to specify a remote location.  At
this point only "logs" upload cleares and creates directories on the fly.

If you want to script interactive prompts you may use the confirm target
to operate yes or no questions before an action:

    # make info confirm dvd

To add arbitrary plugins from an external location into an image you can
use the following:

    # make custom-<image> ADDITIONS="an-existing-plugin path/to/extra/plugin"

Last but not least, a rebuild of YETIsense core and plugins on package
sets is invoked using:

    # make hotfix[-<step>]

The default hotfix run is a non-destructive rebuild pass for missing
plugins and core packages which also signs the existing packages.

You can also do a full rebuild using "core" or "plugins".  The "ports"
step, however, will automatically rebuild mismatching and missing ports.

Any other argument (or list of arguments separated by comma) will be
treated as individual packages to be rebuilt by their matching steps.
