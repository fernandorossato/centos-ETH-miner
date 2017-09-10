##########################################################################
###########                                                              #
########### Minerador de ETH                                             #
########### Desenvolvido por Fernando Rossato <fer.rossato@gmail.com>    #
###########                                                              #
##########################################################################
# Esse minerador é uma junção de alguns scripts com alguns que criei para automatizar a mineração de ETH com o Claymore's no CentOS 7. Não tenho intenção de vender ou dar suporte nesses scripts, modifique a vontade por sua conta em risco
# Eu instalei a versão do CentOS 7 Gnome live e o drive da nvidia.
# Para instalar o Drive da Nvidia e preparar o sistema faça o seguinte:
# Baixe o drive do site da Nvidia
# Execute:
setenforce 0
# Edite o /etc/selinux/config e set SELINUX=disabled
yum update
reboot

# Depois siga com:
yum -y groupinstall "GNOME Desktop" "Development Tools"
yum -y install epel-release
yum -y install kernel-devel
systemctl enable sshd
systemctl set-default multi-user.target
yum -y install dkms

# Edite /etc/default/grub. Adicione a string abaixo em “GRUB_CMDLINE_LINUX”
rd.driver.blacklist=nouveau nouveau.modeset=0

# Aplique as mudanças no grub
grub2-mkconfig -o /boot/grub2/grub.cfg

# Crie o arquivo /etc/modprobe.d/blacklist.conf e adcione:
blacklist nouveau
    
# Faça um Backup do seu initramfs gere um novo
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
dracut /boot/initramfs-$(uname -r).img $(uname -r)

reboot

# Siga para instalação do drive
bash NVIDIA-Linux-x86_64-XXX.XX.run

# Local dos arquivos (Diretorio /root)
# A pasta claymore e os arquivos nvidia-overclock_x11.sh e settings.conf devem estar em /root

## Passo 1 - Copiar scripts
Copie os scripts abaixo para /usr/bin
miner-init
miner-start
miner-stop
miner-monitor
miner-status
nvidia-status

# Passo 2 - Identificar e configurar as placas
Execute:
miner-init

# Passo 3 - Verificar Placas
Execute:
nvidia-status

# Passo 4 - Configurar
Edite:
settings.conf

# Passo 5 - Ativar minerador na inicialização (Depois de 120s ou o tempo que vc achar melhor, 2min é bom tempo para desativar o script caso tenha algum problema)
# Esse passo é para ativar o minerador na inicialização, recomendo fazer alguns testes antes de fazer isso!!
Execute:
crontab -e

Cole:
@reboot sleep 120 && /usr/bin/miner-start

# Passo 6 - Minerar
Execute:
miner-start

# Passo 7 - Monitorar
Execute:
miner-monitor

# O Claymore's também tem um monitoramento via navegador caso vc queira visualizar detalhadamento
Acesse pelo navegador http://IP_MINEIRO:3333

# Se precisar parar (Você não vai querer :))
miner-stop

#####################################################################################################################
# Leituras adicionais, não precisa executar os comandos abaixo, são apenas referencias que usei para fazer os scripts, caso tenha algum problem talvez esses links e comandos podem te ajudar.

# Ativar overclock
nvidia-xconfig -a --cool-bits=28 --allow-empty-initial-configuration

#
https://github.com/Cyclenerd/ethereum_nvidia_miner/tree/master/files

######################################################### // ##################################################
https://devtalk.nvidia.com/default/topic/981655/fan-speed-on-headless-linux-machine-without-performance-loss/?offset=3
Hi,

thanks, for pascal there is no hack in linux yet.

I came across this post:

https://devtalk.nvidia.com/default/topic/789888/set-fan-speed-without-an-x-server-solved-/

It works, one can set the fan speed as desired, and then kill the X server again.

Recommended to run this one first:
nvidia-xconfig --enable-all-gpus --separate-x-screens --allow-empty-initial-configuration

All in one script:

X :0 &
sleep 5
nvidia-settings -a "[gpu:0]/GPUFanControlState=1"
nvidia-settings -a "[gpu:1]/GPUFanControlState=1"
nvidia-settings -a "[gpu:2]/GPUFanControlState=1"
nvidia-settings -a "[gpu:3]/GPUFanControlState=1"
nvidia-settings -a "[fan:0]/GPUTargetFanSpeed=100"
nvidia-settings -a "[fan:1]/GPUTargetFanSpeed=100"
nvidia-settings -a "[fan:2]/GPUTargetFanSpeed=100"
nvidia-settings -a "[fan:3]/GPUTargetFanSpeed=100"
killall Xorg