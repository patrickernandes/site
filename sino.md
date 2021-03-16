---
layout: page
title: Sino
permalink: /sino/
description: > # this means to ignore newlines until "show_exerpts:"
  S I N O - LiveCD sistema para virtualização.
---

# S I N O  

Um sistema Linux em LiveCD para virtualização com XEN.  
É baseado no **Debian GNU/Linux** com *hypervisor* XEN e gerenciador de discos LVM.  

Você pode inicializar o sistema sem modificar nenhum arquivo no disco rígido, não sendo necessário instalá-lo.  
Tem suporte de inicialização por bios legacy ou UEFI.  

Todo o trabalho se encontra em desenvolvimento, sujeito a mudanças e correções, então teremos a honra de receber qualquer sugestão ou dúvidas via e-mail **suporte@ernandes.info**  
&nbsp;

Obrigado,

----

## Introdução:

A ideia de criar este sistema para virtualização foi devido a dificuldade de realizar um *rollback* de uma instalação após o sistema apresentar problemas depois de um *upgrade*.  
SINO é uma "image" ISO que funciona como LiveCD, então você não precisa instalá-lo, apenas gravá-lo em um pendrive para *boot*. Ele não possui interface gráfica, seu gerenciamento é todo via *linha de comando*.  
Sempre que for necessário realizar um upgrade, uma *image* nova dele é disponibilizada e você deverá gravá-lo em um novo pendrive.  
Caso houver algum problema com a nova versão, basta você retornar a utilizar o pendrive anterior.  
Quanto as VMs, todos os dados devem ser armazenados em volumes nos discos locais, utilizando o gerenciador LVM.  

![Sino!](/sino/sino_logon.png "Sino acesso..")  
&nbsp;

## Arquivos:

Segue o link para downloads das *ISOs* e seus respectivos *checksum*. Todos os arquivos estão em uma única pasta:

Arquivos: [pasta](https://www.dropbox.com/sh/9hip5a385kqctar/AAAi8raYbK24QyQPASG47vtta?dl=0){:target="_blank"}   
Última versão: sino-20210316.iso  
Changelog: [arquivo](http://ernandes.info/sino/ChangeLog.txt){:target="_blank"}    
Última alteração: 16/03/2021  
&nbsp;

## Gravar image ISO:

Após baixar a image SINO, será necessário gravá-lo em um pendrive. Exemplo:
```
dd if=sino-20201112.iso of=/dev/sdX bs=1M    
sync
```
* /dev/sdX deve ser alterado de acordo com seu dispositivo USB.  

SINO tem suporte a *boot* por bios legacy ou UEFI.  
&nbsp;

## Inicialização:

Ao iniciar o *boot* pelo pendrive, vai ser apresentado 3 opções de inicialização:  

**SINO** - *boot* normal.  
**SINO on ram** - carrega o sistema na memória RAM.  
**SINO advanced** - em caso de problemas com vídeo framebuffer.   
**SINO advanced on ram** - carregamento na memória RAM sem video em framebuffer.  

Recomendo em ambiente de produção, utilizar a opção **"SINO on ram"**, pois com o sistema carregado na memória RAM será mais rápido e ainda oferecer a possibilidade de remover o pendrive.   
&nbsp;

## Acesso:

Por padrão, o acesso é feito através do usuário *root* e senha *root*.  
Ainda é possível realizar acesso remoto via *ssh*.  
&nbsp;

## Rede:

O processo de *boot* possibilita que o sistema configure automaticamente seu IP, em caso dele estar em uma rede com *dhcp* habilitado.  
Por padrão, a interface de rede utilizada é *xenbr0*, que atua como *bridge* para as máquinas virtuais.  
&nbsp;

## Disco:

Discos locais devem ser utilizados para armazenamento das configurações das maquinas virtuais e seus discos.  
Para isso, vamos utilizar o gerenciador de volumes LVM.  

Vamos usar como exemplo, um disco de 120GB como *sda*:  

*1* - criar uma partição.  
*2* - partição deverá ser tipo "Linux LVM".  

* como saída, teremos a partição "/dev/sda1" de 120GB.  

Agora, vamos criar o grupo de volumes chamado **xvg** que será utilizado para armazenar os volumes lógicos. Por padrão, SINO utiliza o grupo LVM de nomenclatura **xvg**.  
Mas antes, vamos criar um volume físico:  
```
pvcreate /dev/sda1
```

Criando o grupo de volumes *xvg*:  
```
vgcreate xvg /dev/sda1
```

Com o grupo criado, vamos criar um volume denominado **xvol** de 20G para armazenar arquivos ISOs e as configurações das máquinas virtuais. Também como padrão, SINO utiliza o volume de nomenclatura **xvol** para armazenamento permanente.     
```
lvcreate -n xvol -L 20G xvg
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

## Swap:

Para melhorar o desempenho de memória RAM, vamos adicionar um volume para *swap*. Aqui, como exemplo, vamos criar um volume de 6GB:
```
lvcreate -n swap -L 6G xvg
```

Vamos formatar o volume *swap*:
```
mkswap -L swap /dev/xvg/swap
```

Agora, ativar o volume:
```
swapon /dev/xvg/swap
```

Com isso, teremos uma partição em *swap* para auxiliar a memória:  

![Swap](/sino/swap.png "Sino memória swap.")    
&nbsp;

## Máquinas virtuais:

Para o processo de virtualização, o XEN disponibiliza dois modos para virtualizar uma máquina virtual, o **PV** e o **HVM**:  

* PV - quando apenas uma parte do hardware é virtualizado. A outra parte, utiliza o hardware físico.  

* HVM - todo hardware é virtualizado. Todo o hardware para máquina virtual tem que ser virtualizado, uma virtualização completa.  

Para *PV*, podemos criar VMs Debian, pois aproveita partes do sistema SINO, já que o mesmo é baseado no Debian.  
Para *HVM*, já é possível criar maquinas virtuais Linux e também Windows.  

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
Você pode utilizar os editores *vim* ou *nano* para criar ou alterar arquivos.
Abaixo, temos a configuração proposta:  

``` 
builder='hvm'    
type='hvm' 
arch='x86_64'  
viridian=1  
name='windows'  
memory=4096  
vcpus=4  
acpi=1  
vif=[ 'bridge=xenbr0, model=e1000' ]  
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

Antes de criar a VM e iniciá-la, devemos ajustar o arquivo de de configuração de acordo com suas necessidades e depois:  
```
xl create /srv/windows.cfg  
```

* Não esqueça da image ISO de instalação do Windows.  

A VM pode ser acessada remotamente via algum aplicativo cliente para VNC.  
Para melhor desempenho em VMs Windows, sugiro instalar o pacote de drivers *XCP-ng Windows PV Tools*.  
Novos drivers serão instalados, com isso seu desempenho tem uma melhora significativa.  
Para baixar o pacote, segue o link:  

[https://github.com/xcp-ng/win-pv-drivers/releases](https://github.com/xcp-ng/win-pv-drivers/releases)  

Para criar uma VM linux, podemos usar o modelo de configurações abaixo:
```
builder='hvm'  
type='hvm'  
arch='x86_64'  
name='linux'  
memory=2048  
vcpus=2  
acpi=1  
vif=[ 'bridge=xenbr0, model=virtio-net' ]  
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

## Material de apoio:

Alguns links com informações para ajudar com os gerenciadores LVM e XEN:  

Para uso do XEN, temos os links abaixo:  
[https://xenproject.org](https://xenproject.org)  
[https://wiki.xenproject.org/wiki/Main_Page](https://wiki.xenproject.org/wiki/Main_Page)  
[https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide](https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide)

Para LVM:  
[https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))  
[https://wiki.archlinux.org/index.php/LVM](https://wiki.archlinux.org/index.php/LVM)    
[https://bnuti.com.br/como-utilizar-o-lvm-no-linux](https://bnuti.com.br/como-utilizar-o-lvm-no-linux)

&nbsp;

* * *
<font size="2">2019 - 2020 - S I N O.  </font>  
<font size="2">| Linux® is a registered trademark of Linus Torvalds.</font>  
<font size="2">| Windows® is a trademark of Microsoft Corporation.</font>