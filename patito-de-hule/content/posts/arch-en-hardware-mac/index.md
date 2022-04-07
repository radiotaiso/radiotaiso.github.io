+++
title = "A veces creo que es cierto lo que dicen de linux"
description = "esto donde se ve o como?"

date = 2022-04-05

# If set, this slug will be used instead of the filename to make the URL.
# The section path will still be used.
slug = "arch-en-mac"
+++

# Manjaro en una mac?
No soy para nada un experto en mac o linux, esto no es un tutorial, primer aviso.

Estas son las cosas que he descubierto poniendole manjaro a esta madre, esta madre siendo una:

```
Host: MacBookPro14,1
CPU: Intel i7-7660U (4) @ 4.000GHz 
GPU: Intel Iris Plus Graphics 640 
Memory: 16gb
```

## Uhh pero por que?

Catorce coma uno, se traduce en que es una [Macbook Pro Mid 2017](https://support.apple.com/kb/SP754?locale=en_US) de esas del teclado horrible, pero muy afortunada, de todas las macbook pros alla afuera [el estado de linux](https://github.com/Dunedan/mbp-2016-linux#current-status) ese modelo es uno de los dos donde funciona el wifi.


Manjaro solo fue un chiste al principio, no podia creer que una experiencia de instalacion con arch podria ser mucho menos dolorosa que ese instalador de Debian.


## El primer crux

El driver de sonido ha sido raro, mi problema principal es arch (baia baia quien lo iba a pensar) y los `linux-headers`, creo que es importante hacer mencion que no son lo mismo que `linux-api-headers` y tenemos que instalar la version que corresponde al kernel `uname -r` me dice que tengo `5.15.28-1-MANJARO`, existen paquetes para distintas versiones de kernel.

El [repo del driver](https://github.com/davidjo/snd_hda_macbookpro) son dos noticias una buena y una mala, la buena es que tiene un instalador en forma de script `.sh` y estoy 99% seguro que se deberia de poder instalar en arch, la mala es que no considera arch en su instalador.


Hora de desempolvar las habilidades de bash, el instalador falla por que no sabe donde buscar el paquete de `linux515-headers` que recien instalamos, long story short, en arch viven en `/usr/lib/modules/` lo que resulta en el codigo del instalador editado:

```sh
build_dir='build'
update_dir="/lib/modules/$(uname -r)/updates"
patch_dir='patch_cirrus'
hda_dir="$build_dir/hda-$kernel_version"

[[ ! -d $update_dir ]] && mkdir $update_dir
[[ ! -d $build_dir ]] && mkdir $build_dir
[[ -d $hda_dir ]] && rm -rf $hda_dir

if [ -d /usr/src/linux-headers-$(uname -r) ]; then
	# Debian Based Distro
	:
elif [ -d /usr/src/kernels/$(uname -r) ]; then
	# Fedora Based Distro
	:
elif [ -d /usr/lib/modules/$(uname -r) ]; then
	# Arch Based Distro
	:
else
	echo "linux kernel headers not found in /usr/src:"
	echo "Debian (eg Ubuntu): /usr/src/linux-headers-$(uname -r)"
	echo "Fedora: /usr/src/kernels/$(uname -r)"
	echo "assuming the linux kernel headers package is not installed"
	echo "please install the appropriate linux kernel headers package:"
	echo "sudo apt install linux-headers-$revpart3"

	exit 1

fi
```

Justo debajo del `elif` de *fedora*, agregue otro elif con el path correcto para arch, esto hace que el script se ejecute "correctamente" pero el gui de pulseaudio sigue sin ver nada, mi sospecha es que el script pues en algun otro punto se pierda por que no encuentra las cosas solo buscando en default debian y fedora.


!["jojo's bizzarre adventure to be continued manga arrow"](jojo-arrow.png)