# Introdução ao Vagrant (Windows + VirtualBox)

Este guia é um tutorial **básico** de instalação e uso do **Vagrant** em **Windows**, usando **VirtualBox** como provedor.  
Inclui exemplos de:

- Instalar Vagrant  
- Conceitos básicos
- Subir uma VM **Linux** da SecDay
- Subir uma VM **Windows** da SecDat  
- Acessar VMS

---

## 1. O que é o Vagrant?

O **Vagrant** é uma ferramenta para criar e gerenciar máquinas virtuais de forma simples e automatizada, usando arquivos de configuração de texto (`Vagrantfile`).

Você usa o Vagrant para:

- Criar ambientes de desenvolvimento  
- Subir e destruir VMs rapidamente  
- Compartilhar configuração com outras pessoas (apenas enviando o `Vagrantfile`)

Neste guia vamos usar:

- **Host:** Windows  
- **Provider (hipervisor):** VirtualBox  
- **Guest (VMs):** Linux e Windows

---

## 2. Pré-requisitos

1. **Windows 10 ou 11** (64 bits)
2. Acesso de administrador para instalar programas
3. Suporte a virtualização ativado na BIOS/UEFI  
   - Normalmente chama-se **Intel VT-x**, **AMD-V** ou “Virtualization Technology”
1. **VirtualBox** Instalado

---

## 3. Instalação

### 3.1. Instalar o Vagrant

1. Acesse o site oficial do [Vagrant](https://developer.hashicorp.com/vagrant/install#windows).  
2. Baixe o instalador para **Windows**.  
3. Execute o instalador (`.msi`) e siga os passos padrão (ao final, você precisará reiniciar sua maquina).  
4. Após instalar e reiniciar a maquina, abra o **PowerShell** ou **Prompt de Comando** e verifique:

```powershell
vagrant --version
````

Se aparecer a versão (ex: `Vagrant 2.x.x`), está instalado corretamente.

---

## 4. Conceitos básicos do Vagrant

- **Box:** imagem base da VM (semelhante a um “template”).
    
- **Vagrantfile:** arquivo de configuração que descreve a VM.
    
- **Provider:** tecnologia de virtualização (no nosso caso, VirtualBox).
    

Comandos principais:

```bash
vagrant init    # Cria Vagrantfile
vagrant up      # Sobe (cria/inicia) a VM
vagrant halt    # Desliga a VM
vagrant reload  # Reinicia a VM aplicando mudanças no Vagrantfile
vagrant destroy # Remove a VM
vagrant status  # Mostra o estado da VM
```

> Todos esses comandos são executados na pasta onde está o `Vagrantfile`.

---

## 5. Criando uma VM Linux (exemplo com Ubuntu)

### 5.1. Criar a pasta do projeto

No Windows (PowerShell ou CMD):

```powershell
mkdir C:\vagrant\linux-ubuntu
cd C:\vagrant\linux-ubuntu
```

### 5.2. Inicializar o Vagrantfile

```powershell
vagrant init secday/ubuntu
```

Isso cria um `Vagrantfile` básico usando a box `ubuntu/focal64`.

### 5.3. Exemplo de Vagrantfile para Linux (SSH) da SecDay

Abra o `Vagrantfile` em um editor de texto (Notepad++, VS Code, etc.) e deixe parecido com:

```ruby
Vagrant.configure("2") do |config|
  # Box base
  config.vm.box = "SecDay/ubuntu"
  config.vm.hostname = "ubuntu.local"
  config.vm.box_version = "1.0.0"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 22, host: 2221, protocol: "tcp", auto_correct: true
  config.vm.provision "shell",
    inline: "apt-get update && apt-get install virtualbox-guest-additions-iso -y && reboot"

  # Nome da VM no VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.name = "Ubuntu-SecDay"
    vb.memory = "1024"
    vb.cpus = 1
    vb.customize ["modifyvm", :id, "--groups", GRUPO_VBOX]
  end

end
```

### 5.4. Subir a VM Ubuntu da SecDay

Na pasta do projeto:

```powershell
vagrant up
```

O Vagrant irá:

1. Baixar a box (primeira vez)
    
2. Criar a VM no VirtualBox
    
3. Iniciar a VM
    

### 5.5. Acessar a VM via SSH

```powershell
vagrant ssh
```

Isso já entra na VM como usuário padrão (`vagrant`), sem precisar configurar chave ou senha manualmente.

Para sair da VM:

```bash
exit
```

### 5.5. Destruir a VM

```
vagrant destroy
```

Isso irá remover a VM criada e todos os seus arquivos. Uma mensagem de confirmação aparecerá na tela; selecione `y` para prosseguir com a exclusão.

---

## 6. Criando uma VM Windows da SecDay

### 6.1. Criar a pasta do projeto Windows

```powershell
mkdir C:\vagrant\windows-server
cd C:\vagrant\windows-server
```

### 6.2. Inicializar com a box Windows 

```powershell
vagrant init SecDay/windows_2022
```

### 6.3. Exemplo de Vagrantfile para o Windows

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "SecDay/windows_2022"
  config.vm.hostname = "windows2022"
  config.vm.network "private_network", ip: "192.168.56.11"
  config.winrm.timeout = 600
  config.vm.network "forwarded_port", guest: 3389, host: 33891, protocol: "tcp", auto_correct: true
  config.vm.communicator = "winrm"

  # Configuração do VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.name = "windows_2022-SecDay"
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--groups", GRUPO_VBOX]
  end
end
```

### 6.4. Subir a VM Windows

```powershell
vagrant up
```

Se tudo estiver configurado corretamente, o Vagrant irá subir a VM Windows no VirtualBox e se comunicar com ela via **WinRM**.

### 6.5. Destruir a VM

```
vagrant destroy
```

Isso irá remover a VM criada e todos os seus arquivos. Uma mensagem de confirmação aparecerá na tela; selecione `y` para prosseguir com a exclusão.

---

## 7. Acessando a VM Windows via Remote Desktop

A box Windows da SecDay vem com o RDP habilitado e você pode acessar pela porta de RDP:

No `Vagrantfile`, por exemplo:

```ruby
config.vm.network "forwarded_port", guest: 3389, host: 33891, auto_correct: true
```

Depois de subir a VM:

1. Abra **Conexão de Área de Trabalho Remota** no Windows (`mstsc`).
    
2. Conecte em: `localhost:33891`
    
3. Use o usuário/senha configurados na box (ex.: `vagrant` / `vagrant`).
    
---

## 9. Comandos úteis do Vagrant

Na pasta do projeto (onde está o `Vagrantfile`):

```bash
vagrant status   # Ver estado da VM
vagrant halt     # Desligar a VM
vagrant reload   # Reiniciar a VM (aplicando alterações do Vagrantfile)
vagrant destroy  # Destruir a VM (atenção: perde tudo dentro dela)
```

Para atualizar uma box:

```bash
vagrant box outdated
vagrant box update
```

---
## 10. Dicas finais

- Sempre execute os comandos do Vagrant na pasta onde está o `Vagrantfile`.
    
- Se der erro, rode:
    
    ```bash
    vagrant status
    vagrant up --debug
    ```
    
    para ver mais detalhes do problema.
    
- Verifique se o **VirtualBox** está atualizado e compatível com a versão do Vagrant.
    
- Evite abrir/alterar a VM diretamente pelo VirtualBox enquanto o Vagrant está gerenciando a máquina.

- Você pode acessar diretamente a documentação para mais detalhes [aqui](https://developer.hashicorp.com/vagrant/docs)
    
