---
layout: page
title: Sino
permalink: /sino/
description: > # this means to ignore newlines until "show_exerpts:"
  S I N O - LiveCD sistema para virtualização.
---

# S I N O  

Um sistema Linux em LiveCD para virtualização com XEN.  
É baseado no **Debian GNU/Linux Buster** com *hypervisor* XEN e gerenciador de discos LVM.  

Você pode inicializar o sistema sem modificar nenhum arquivo no disco rígido, nao é necessário instalá-lo.  
Tem suporte de inicialização por bios legacy ou UEFI.  

Todo o trabalho se encontra em desenvolvimento, sujeito a mudanças e correções, então teremos a honra de receber qualquer sugestão ou dúvidas via e-mail **suporte@ernandes.info**  
&nbsp;

Obrigado,
 
----

## Indrodução:
 
A ideia de criar este sistema para virtualização foi devido a dificuldade de realizar um *rollback* de uma instalação após o sistema apresentar problemas depois de um *upgrade*.  
SINO é uma "image" ISO que funciona como LiveCD, então você não precisa instalá-lo, apenas gravá-lo em um pendrive para *boot*. Ele não possui interface gráfica, seu gerenciamente é tudo via *command line*.  
Sempre que for necessário realizar um upgrade, uma *image* nova dele é disponibilizada e você deverá gravá-lo em um novo pendrive.  
Caso houver algum problema com a nova versão, basta você retornar a utilizar o pendrive anterior.  
Quanto as *VMs*, todos os dados devem ser armazenados em volumes nos discos locais, utilizando o gerenciador LVM.  

![Sino!](/sino/sino_logon.png "Sino acesso..")
&nbsp;

## Changelog:

Segue o link com os dados referente ao changelog: 

Changelog: [arquivo](https://www.dropbox.com/s/wbx9jg0agjqf0ls/ChangeLog.txt?dl=0)    
Última alteração: 12/11/2020
&nbsp;


## Arquivos:

Segue o link para downloads das *ISOs* e seus respectivos *checksum*. Todos os arquivos estão em uma única pasta:

Arquivos: [pasta](https://www.dropbox.com/sh/9hip5a385kqctar/AAAi8raYbK24QyQPASG47vtta?dl=0)   
Última versão: sino-20201112.iso  
&nbsp;

## Gravar "image":

Após baixar a image SINO, será necessário gravá-lo em um pendrive.

`# dd if=sino-* of=/dev/sdX`    
`# sync`
    
SINO tem suporte a *boot* por bios legacy ou UEFI.  
&nbsp;

## Inicialização:

Ao iniciar o *boot* pelo pendrive, vai ser apresentado 3 opções de inicialização:  

**SINO** - *boot* normal.  
**SINO on ram** - carrega o sistema na memoria RAM.  
**SINO advanced** - em caso de problemas com video.  

Recomendo em ambiente de produção, utilizar a opção **SINO on ram**, pois com o sistema carregado na memória RAM será mais rapido e ainda oferecer a possibilidade de remover o pendrive.   
&nbsp;

## Acesso:

Por padrão, o acesso é feito através do usuário *root* e senha *root*.  
Ainda é possivel realizar acesso remoto via *ssh*.  
&nbsp;

## Rede:

O processo de *boot* possibilita que o sistema configure automaticamente seu IP, em caso dele estar em uma rede com *dhcp* habilitado.  
Por padrão, a interface de rede utilizada é *br0*, que também atua como *bridge* para as máquinas virtuais.  
&nbsp;

## Disco:

Discos locais devem ser utilizados para armazenamento das configurações das maquinas virtuais e seus discos.  
Para isso, vamos utilizar o gerenciador de volumes LVM.  

Vamos usar como exemplo, um disco de 20GB, como *sda*:  

*1* - criar uma partição.  
*2* - partição deverá ser tipo "Linux LVM".  

* como saída, teremos a partição "/dev/sda1" de 20GB.  

Agora, vamos criar o grupo de volumes chamado **xvg** que será utilizado para armazenar os volumes lógicos.  
Mas antes, vamos criar um volume físico:  
```
pvcreate /dev/sda1
```

Criando o grupo de volumes *xvg*:  
```
vgcreate xvg /dev/sda1
```

Com o grupo criado, vamos criar um volume denominado **xvol** de 10G para armazenar arquivos ISO's e as configurações das máquinas virtuais.  
```
lvcreate -n xvol -L 10G xvg
```

Devemos formatar o volume *xvol* com *ext4*:  
```
mkfs.ext4 /dev/xvg/xvol
```

Com a configuração de armazenamento pronta, vamos iniciar o script de disco que carrega o volume *xvol* na pasta "/srv":  
```
srv start
```

Qualquer material que temos que manter a salvo, deve ser armazenado na pasta "/srv".  
&nbsp;

## Máquinas virtuais:

Para o processo de virtualização, o XEN disponibiliza dois modos para virtualizar uma máquina virtual, o **PV** e o **HVM**:  

* PV - quando apenas uma parte do hardware é virtualizado. A outra parte, utiliza o hardware físico.  

* HVM - todo hardware é virtualizado. Todo o hardware para máquina virtual tem que ser virtualizado, uma virtualização completa.  

Para *PV*, podemos criar VM's Debian, pois aproveita partes do sistema SINO, já que o mesmo é baseado no Debian.  
Para *HVM*, já é possivel criar maquinas virtuais Linux e também Windows.  

### PV

Como exemplo, vamos criar uma máquina virtual Debian em *PV*:
```
xen-create-image --hostname=teste1 --dist=buster --pygrub --vcpus=1 --memory=512mb --lvm=xvg --size=2g --dhcp
```

Uma VM de hostname "teste1" será criada.
Para acessar a mesma:
```
xl console teste1
```

Assim temos nossa primeira VM criada, em modo PV.  

### HVM

Com virtualização HVM, podemos criar um ambiente todo virtualizado,  máquinas Linux e Windows.  
É o modo mais recomendado para criar suas máquinas virtuais.   
 
Como exemplo, temos o arquivo 'windowsexample.cfg' que serve de modelo parara criar uma VM Windows.  
Voce pode utilizar os editores *vim* ou *nano* para criar ou alterar arquivos.
Abaixo, temos a configuração proposta:  
``` 
builder='hvm'    
type='hvm' 
arch='x86_64'  
viridian=1  
name='windows'  
memory=4096  
vcpus=2  
acpi=1  
vif=[ 'bridge=br0, model=e1000' ]  
disk=[ 'phy:/dev/xvg/windows-disk0,hda,w', 'file:/srv/windows.iso,hdc:cdrom,r' ]  
boot='dc'  
sdl=0  
vnc=1  
vnclisten='0.0.0.0'  
stdvga=0  
serial='pty'  
usb=1  
usbdevice='tablet'  
keymap='pt-br'  
on_poweroff='destroy'  
on_reboot  ='restart'  
on_crash   ='restart'  
```

Para iniciar a criação da VM, antes vamos criar seu disco, um volume LVM:
```
lvcreate -n windows-disk0 -L 60G xvg
```

Agora, vamos criar nossa VM:  
```
cp /root/windowsexample.cfg /srv/windows.cfg  
```

Antes de criar a VM e iniciá-la, é possivel ajustar o arquivo de acordo com suas necessidades e depois:  
```
xl create /srv/windows.cfg  
```

* Não esqueça da image ISO de instalação do Windows.  

A Vm pode ser acessada remotamente via algum aplicativo cliente para VNC. 

Para criar uma VM linux, podemos usar o modelo de configurações abaixo:
```
builder='hvm'  
type='hvm'  
arch='x86_64'  
name='linux'  
memory=2048  
vcpus=2  
acpi=1  
vif=[ 'bridge=br0, model=virtio-net' ]  
disk=[ 'phy:/dev/xvg/linux-disk0,hda,w', 'file:/srv/linux.iso,hdc:cdrom,r' ]  
boot='dc'  
sdl=0  
vnc=1  
vnclisten='0.0.0.0'  
stdvga=0  
serial='pty'  
usb=1  
usbdevice='tablet'  
keymap='pt-br'  
on_poweroff='destroy'  
on_reboot  ='restart'  
on_crash   ='restart'  
```

Todas as configurações devem ser alteradas de acordo com suas necessidades.    
&nbsp;

## Material de apoio.

Alguns links para ajudar com os gerenciadores LVM e XEN:  

* Para uso do XEN, temos os links abaixo:  
https://xenproject.org  
https://wiki.xenproject.org/wiki/Main_Page  
https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide

* Para LVM:  
https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)  
https://wiki.archlinux.org/index.php/LVM    
https://bnuti.com.br/como-utilizar-o-lvm-no-linux

 