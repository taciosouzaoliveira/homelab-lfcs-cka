# **Guia de Instalação: Vagrant com KVM/Libvirt em Debian/Ubuntu**

Este guia detalha o processo de configuração de um ambiente de desenvolvimento local utilizando Vagrant com o provedor KVM/Libvirt em sistemas baseados em Debian (como Debian 11/12 e Ubuntu 22.04+).

Esta abordagem é uma alternativa mais performática e nativa ao VirtualBox para usuários de Linux.

## **Pré-requisitos**

- Um sistema operacional Debian, Ubuntu ou derivado
- Acesso de superusuário (sudo)
- Virtualização de hardware (VT-x ou AMD-V) habilitada na BIOS/UEFI do seu computador
- Pelo menos 8GB de RAM (recomendado 16GB+ para múltiplas VMs)
- Espaço em disco suficiente para as máquinas virtuais

## **Passo 1: Verificar Suporte à Virtualização**

Antes de começar, verifique se sua CPU suporta virtualização:

bash

```
# Verificar suporte à virtualizaçãoegrep -c '(vmx|svm)' /proc/cpuinfo

# Se retornar 1 ou mais, a virtualização está habilitada# Se retornar 0, você precisa habilitar a virtualização na BIOS/UEFI
```

## **Passo 2: Instalação do KVM e Libvirt**

Primeiro, atualizamos os pacotes do sistema e instalamos o KVM, o daemon Libvirt e outras ferramentas de gerenciamento necessárias.

bash

```
# Atualiza a lista de pacotessudo apt update

# Instala os pacotes essenciais para KVM, Libvirt e gerenciamento de virtualizaçãosudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst libvirt-daemon virt-manager

# Instala dependências adicionais úteissudo apt install -y dnsmasq qemu-utils ebtables
```

## **Passo 3: Configuração de Permissões**

Para que o Vagrant possa controlar o Libvirt sem a necessidade de sudo para cada comando, adicionamos nosso usuário ao grupo libvirt.

bash

```
# Adiciona o seu usuário atual ao grupo 'libvirt'sudo adduser $USER libvirt

# Adiciona também ao grupo kvm para acesso completosudo adduser $USER kvm
```

⚠️ **Importante**: Após executar o comando acima, você precisa fazer logout e login novamente ou reiniciar o seu sistema para que a nova associação de grupo tenha efeito. Para aplicar em sua sessão atual sem reiniciar, você pode usar o comando `newgrp libvirt`, mas o logout/login é a forma mais garantida.

## **Passo 4: Instalação do Vagrant**

Vamos instalar a versão mais recente do Vagrant diretamente do repositório oficial da HashiCorp para garantir que estamos atualizados.

bash

```
# 1. Adiciona a chave GPG oficial da HashiCorpwget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# 2. Adiciona o repositório da HashiCorpecho "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# 3. Atualiza a lista de pacotes e instala o Vagrantsudo apt update
sudo apt install vagrant -y
```

## **Passo 5: Instalação do Plugin Vagrant-Libvirt**

Este plugin é a "cola" que permite ao Vagrant se comunicar com o hypervisor KVM/Libvirt.

bash

```
# Instala dependências necessárias para o pluginsudo apt install -y build-essential pkg-config libvirt-dev

# Instala o plugin vagrant-libvirt
vagrant plugin install vagrant-libvirt

# Instala também o plugin vagrant-mutate (útil para converter boxes)
vagrant plugin install vagrant-mutate
```

Para verificar se todos os plugins estão instalados corretamente, você pode listar os plugins instalados:

bash

```
vagrant plugin list
```

Você deve ver `vagrant-libvirt` na lista.

## **Passo 6: Configuração Adicional (Opcional)**

### **Configurar rede bridge (para acesso externo às VMs)**

bash

```
# Instalar ferramentas de redesudo apt install -y net-tools

# Verificar interfaces de redeip addr show
```

### **Configurar storage pool padrão**

bash

```
# Verificar pools de storage existentesvirsh pool-list

# Criar um pool de storage personalizado (opcional)mkdir -p ~/virt/images
virsh pool-define-as --name default --type dir --target ~/virt/images
virsh pool-autostart default
virsh pool-start default
```

## **Passo 7: Verificação do Ambiente**

Neste ponto, seu ambiente está pronto. Vamos verificar se tudo está funcionando como esperado.

bash

```
# Verifique a versão do Vagrant
vagrant --version

# Verifique se o serviço libvirtd está ativosudo systemctl status libvirtd

# Verifique se o usuário está nos grupos corretosgroups $USER

# Liste as VMs (a lista deve estar vazia)virsh list --all

# Teste a comunicação com libvirtvirsh --connect qemu:///system capabilities
```

Se todos os comandos executarem sem erros, a instalação foi um sucesso!

## **Passo 8: Teste com uma Box Básica**

Vamos testar o ambiente com uma box simples:

bash

```
# Criar diretório de testemkdir ~/vagrant-test && cd ~/vagrant-test

# Inicializar um Vagrantfile com uma box compatível
vagrant init generic/ubuntu2004

# Modificar o Vagrantfile para usar o provider libvirtcat > Vagrantfile << EOF
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
    libvirt.memory = 1024
    libvirt.cpus = 1
  end
end
EOF

# Iniciar a VM
vagrant up --provider=libvirt
```

## **Como Usar no Projeto do Home Lab**

Agora você está pronto para subir o seu cluster Kubernetes.

Clone o repositório do seu projeto:

bash

```
git clone https://github.com/taciosouzaoliveira/homelab-lfcs-cka.git
cd homelab-lfcs-cka
```

Inicie o ambiente: Como o vagrant-libvirt é o único provedor de virtualização que configuramos, o Vagrant o utilizará por padrão.

bash

```
vagrant up
```

Se por algum motivo você tiver outros provedores, pode forçar o uso do libvirt:

bash

```
vagrant up --provider=libvirt
```

## **Solução de Problemas Comuns**

### **Erro de permissões**

bash

```
# Se encontrar erros de permissão, reinicie o serviço libvirtsudo systemctl restart libvirtd

# Verifique as permissões do socketsudo chmod 666 /var/run/libvirt/libvirt-sock
```

### **Plugin não instala**

bash

```
# Se o plugin vagrant-libvirt não instalarsudo apt install -y ruby-dev build-essential
vagrant plugin uninstall vagrant-libvirt
vagrant plugin install vagrant-libvirt
```

### **Problemas de rede**

bash

```
# Reiniciar serviços de redesudo systemctl restart libvirtd
sudo systemctl restart virtlogd
```

## **Comandos Úteis de Gerenciamento**

- **Verificar o status das VMs**: `vagrant status`
- **Acessar uma VM via SSH**: `vagrant ssh <nome-da-vm>` (ex: `vagrant ssh k8s-master`)
- **Desligar as VMs**: `vagrant halt`
- **Ligar as VMs**: `vagrant up`
- **Destruir e remover completamente as VMs**: `vagrant destroy -f`
- **Ver logs do Vagrant**: `vagrant up --debug`
- **Listar boxes instaladas**: `vagrant box list`
- **Remover uma box**: `vagrant box remove NOME_DA_BOX`

## **Gerenciamento Avançado com virsh**

- **Listar todas as VMs**: `virsh list --all`
- **Ver detalhes de uma VM**: `virsh dominfo NOME_DA_VM`
- **Desligar uma VM**: `virsh shutdown NOME_DA_VM`
- **Forçar desligamento**: `virsh destroy NOME_DA_VM`
- **Iniciar uma VM**: `virsh start NOME_DA_VM`

Seu ambiente agora está configurado e pronto para uso com Vagrant e KVM/Libvirt!
