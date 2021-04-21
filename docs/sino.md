---
layout: page
title: Sino
permalink: /sino/
description: > # this means to ignore newlines until "show_exerpts:"
  S I N O - LiveCD sistema para virtualização.
---

# S I N O  

Um sistema Linux em LiveCD para virtualização com QEMU/KVM.  
É baseado no **Debian GNU/Linux** com *hypervisor* KVM, emulador QEMU e gerenciador de volumes LVM.  

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

![Sino!](/sino/sino_acesso.png "Sino acesso..")  
&nbsp;

## Arquivos:

Segue o link para downloads das *ISOs* e seus respectivos *checksum*. Todos os arquivos estão em uma única pasta:

Arquivos: [pasta](https://www.dropbox.com/sh/9hip5a385kqctar/AAAi8raYbK24QyQPASG47vtta?dl=0){:target="_blank"}   
Última versão: sino-20210420.iso  
Changelog: [arquivo](http://ernandes.info/sino/ChangeLog.txt){:target="_blank"}    
Última alteração: 20/04/2021  
&nbsp;

## Gravar image ISO:

Após baixar a image SINO, será necessário gravá-lo em um pendrive. Exemplo:
```
dd if=sino-20210420.iso of=/dev/sdX bs=1M    
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

![Swap](/sino/boot.png "Sino boot menu.")    
&nbsp;

Recomendo em ambiente de produção, utilizar a opção **"SINO on ram"**, pois com o sistema carregado na memória RAM, ele será mais rápido, vai evitar possíveis erros de I/O na porta USB e ainda oferecer a possibilidade de remover o pendrive.   
&nbsp;

## Acesso:

Por padrão, o acesso é feito através do usuário *root* e senha *root*.  
Ainda é possível realizar acesso remoto via *ssh*.  
&nbsp;

## Rede:

O processo de *boot* possibilita que o sistema configure automaticamente seu IP, em caso dele estar em uma rede com *dhcp* habilitado.  
Por padrão, a interface de rede utilizada é *br0*, que atua como *bridge* para as máquinas virtuais.  
&nbsp;

## Disco:

Discos locais devem ser utilizados para armazenamento das configurações das máquinas virtuais e seus discos.  
Para isso, vamos utilizar o gerenciador de volumes LVM.  

Vamos usar como exemplo, um disco de 120GB como *sda*:  

*1* - criar uma partição.  
*2* - partição deverá ser tipo "Linux LVM".  

* como saída, teremos a partição "/dev/sda1" de 120GB.  

Agora, vamos criar o grupo de volumes chamado **lvg** que será utilizado para armazenar os volumes lógicos. Por padrão, SINO utiliza o grupo LVM de nomenclatura **lvg**.  
Mas antes, vamos criar um volume físico:  
```
pvcreate /dev/sda1
```

Criando o grupo de volumes *lvg*:  
```
vgcreate lvg /dev/sda1
```

Com o grupo criado, vamos criar um volume denominado **srv** de 20G para armazenar arquivos ISOs e as configurações das máquinas virtuais. Também como padrão, SINO utiliza o volume de nomenclatura **srv** para armazenamento permanente.  
```
lvcreate -n srv -L 20G lvg
```

Devemos formatar o volume *srv* com *ext4*:  
```
mkfs.ext4 /dev/lvg/srv
```

Com a configuração de armazenamento pronta, vamos iniciar o script de disco que carrega o volume *srv* para a pasta "/srv":  
```
srv start
```

Qualquer material que temos que manter a salvo, deve ser armazenado na pasta "/srv".  
&nbsp;

## Swap:

Para melhorar o desempenho de memória RAM, vamos adicionar um volume para **swap**. Aqui, como exemplo, vamos criar um volume de 4GB, com nome **swap**:
```
lvcreate -n swap -L 4G lvg
```

Vamos formatar o volume *swap*:
```
mkswap -L swap /dev/lvg/swap
```

Agora, ativar o volume:
```
swapon /dev/lvg/swap
```

Com isso, teremos uma partição em **swap** para auxiliar a memória RAM:  

![Swap](/sino/swap.png "Sino swap.")    
&nbsp;

## Máquinas virtuais:

Para o processo de virtualização, no ambiente SINO temos a ferramenta **vm** para administrar as máquinas virtuais (VM).  
Lembrando, toda administração é via *cli*, que pode ser utilizada remotamente via conexão *ssh*.  
Com *vm*, podemos criar discos e máquinas virtuais, administra-las, como também remove-las.  
É possível virtualizar VMs Linux e Windows.  
A seguir, o menu com as opções disponíveis para a ferramenta *vm*:  
```
 vm help

Usage: vm [opções..]

  disk                              - listar os discos.
  disk create <size>                - criar um novo disco.
  disk remove <name>                - remover um disco.
  disk use                          - verificar os discos que estão sendo usados.
  list                              - listar as VMs.
  list <name>                       - listar uma VM especifica.
  create <name>                     - criar uma VM.
  show <name>                       - mostrar detalhes da VM.
  remove <name>                     - remover uma VM.
  set <name> --os <linux,windows>   - definir o sistema operacional da VM.
  set <name> --cpu <value>          - definir o número de vcpu em uma VM.
  set <name> --ram <value>          - definir quantidade de memória RAM da VM.
  set <name> --disk0 <name,null>    - definir o disco da VM.
  set <name> --disk1 <name,null>    - definir um segundo disco para a VM.
  set <name> --iso <pathiso,null>   - definir arquivo 'iso' para a VM.
  set <name> --boot <disk,iso>      - definir a ordem de boot da VM.
  set <name> --vnc <yes,no>         - disponibilizar conexão VNC em uma VM.
  start <name>                      - inicializar a VM.
  stop <name>                       - finalizar a VM.
  stop <name> --force               - finalizar de maneira forçada a VM.
  monitor <name>                    - conectar a console do QEMU para interagir com a VM.
  top                               - monitor de consumo das VMs.
```
&nbsp;

## Criando sua primeira VM:

Com mão na massa vamos criar a primeira VM. Como exemplo, ela vai se chamar 'teste'.  
Mas antes, vamos criar um disco para ela, um volume LVM com tamanho de 5GB:
```
vm disk create 5
```
Temos a saída abaixo:  
![Disk](/sino/vm_disk_create.png "Sino disk create.")  
&nbsp;

Agora, vamos criar a VM:
```
vm create teste
```
Saída:  
![Create](/sino/vm_create.png "Sino create.")  
&nbsp;

Um arquivo com nomenclatura "teste.vm" é criado na pasta */srv*.  
Vamos ver os detalhes da VM:
```
vm show teste
```
![Show](/sino/vm_show.png "Sino show.")  
&nbsp;

Vamos ajustar a VM com alguns parâmetros, como sistema operacional, quantidade de cpu e memória, disco e image ISO de boot. 
```
vm set teste --os linux --cpu 2 --ram 1024
vm set teste --disk0 vdisk-8bsncdkgcva37msoaeg2
vm set teste --iso /srv/firmware-10.9.0-amd64-netinst.iso
```
![Show](/sino/vm_show_configurado.png "Sino show after.")  

Agora, só iniciar a VM:
```
vm start teste
```
![Start](/sino/vm_start.png "Sino start.")    

Para conferir seu funcionamento, só executar:
```
vm list
```
![Started](/sino/vm_started_list.png "Sino started.")  

Com um cliente VNC, você pode se conectar remotamente a VM:
```
vnc <ip_sino>:<vnc_port>
```

Para desligar a VM:
```
vm stop teste
```

Assim temos nossa primeira VM criada!   
Uma coisa a ressaltar, em VMs Windows, fica sugestivo a instalação do agente QEMU através do pacote  **virtio-win**. 
&nbsp;

## Material de apoio:

Alguns links com informações para ajudar com QEMU/KVM e com o gerenciador LVM:  

Para uso do QEMU/KVM, temos os links abaixo:  
[https://qemu-project.gitlab.io/qemu/](https://qemu-project.gitlab.io/qemu/)  
[https://github.com/virtio-win/virtio-win-pkg-scripts](https://github.com/virtio-win/virtio-win-pkg-scripts)  
[https://www.linux-kvm.org/page/Main_Page](https://www.linux-kvm.org/page/Main_Page)

Para LVM:  
[https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))  
[https://wiki.archlinux.org/index.php/LVM](https://wiki.archlinux.org/index.php/LVM)    
[https://bnuti.com.br/como-utilizar-o-lvm-no-linux](https://bnuti.com.br/como-utilizar-o-lvm-no-linux)

&nbsp;

* * *
<font size="2">2019 - 2020 - S I N O.  </font>  
<font size="2">| Linux® is a registered trademark of Linus Torvalds.</font>  
<font size="2">| Windows® is a trademark of Microsoft Corporation.</font>
