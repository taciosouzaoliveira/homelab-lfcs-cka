Guia de Instalação: Vagrant com KVM/Libvirt em Debian/Ubuntu
Este guia detalha o processo de configuração de um ambiente de desenvolvimento local utilizando Vagrant com o provedor KVM/Libvirt em sistemas baseados em Debian (como Debian 11/12 e Ubuntu 22.04+).

Esta abordagem é uma alternativa mais performática e nativa ao VirtualBox para usuários de Linux.

Pré-requisitos
Um sistema operacional Debian, Ubuntu ou derivado.

Acesso de superusuário (sudo).

Virtualização de hardware (VT-x ou AMD-V) habilitada na BIOS/UEFI do seu computador.

Passo 1: Instalação do KVM e Libvirt
Primeiro, atualizamos os pacotes do sistema e instalamos o KVM, o daemon Libvirt e outras ferramentas de gerenciamento necessárias.

Bash

# Atualiza a lista de pacotes
sudo apt update

# Instala os pacotes essenciais para KVM, Libvirt e gerenciamento de virtualização
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst libvirt-daemon virt-manager
Passo 2: Configuração de Permissões
Para que o Vagrant possa controlar o Libvirt sem a necessidade de sudo para cada comando, adicionamos nosso usuário ao grupo libvirt.

Bash

# Adiciona o seu usuário atual ao grupo 'libvirt'
sudo adduser $USER libvirt
⚠️ Importante: Após executar o comando acima, você precisa fazer logout e login novamente ou reiniciar o seu sistema para que a nova associação de grupo tenha efeito. Para aplicar em sua sessão atual sem reiniciar, você pode usar o comando newgrp libvirt, mas o logout/login é a forma mais garantida.

Passo 3: Instalação do Vagrant
Vamos instalar a versão mais recente do Vagrant diretamente do repositório oficial da HashiCorp para garantir que estamos atualizados.

Bash

# 1. Adiciona a chave GPG oficial da HashiCorp
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# 2. Adiciona o repositório da HashiCorp
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# 3. Atualiza a lista de pacotes e instala o Vagrant
sudo apt update
sudo apt install vagrant -y
Passo 4: Instalação do Plugin Vagrant-Libvirt
Este plugin é a "cola" que permite ao Vagrant se comunicar com o hypervisor KVM/Libvirt.

Bash

# Instala o plugin vagrant-libvirt
vagrant plugin install vagrant-libvirt
Para verificar se todos os plugins estão instalados corretamente, você pode listar os plugins instalados:

Bash

vagrant plugin list
Você deve ver vagrant-libvirt na lista.

Passo 5: Verificação do Ambiente
Neste ponto, seu ambiente está pronto. Vamos verificar se tudo está funcionando como esperado.

Bash

# Verifique a versão do Vagrant
vagrant --version

# Verifique se o serviço libvirtd está ativo
sudo systemctl status libvirtd

# Liste as VMs (a lista deve estar vazia)
virsh list --all
Se todos os comandos executarem sem erros, a instalação foi um sucesso!

Como Usar no Projeto do Home Lab
Agora você está pronto para subir o seu cluster Kubernetes.

Clone o repositório do seu projeto:

Bash

git clone https://github.com/taciosouzaoliveira/homelab-lfcs-cka.git
cd homelab-lfcs-cka
Inicie o ambiente: Como o vagrant-libvirt é o único provedor de virtualização que configuramos, o Vagrant o utilizará por padrão.

Bash

vagrant up
Se por algum motivo você tiver outros provedores, pode forçar o uso do libvirt:

Bash

vagrant up --provider=libvirt
Comandos Úteis de Gerenciamento
Verificar o status das VMs: vagrant status

Acessar uma VM via SSH: vagrant ssh <nome-da-vm> (ex: vagrant ssh k8s-master)

Desligar as VMs: vagrant halt

Ligar as VMs: vagrant up

Destruir e remover completamente as VMs: vagrant destroy -f
