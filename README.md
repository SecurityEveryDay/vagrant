# Introdução ao Vagrant (Windows + VirtualBox)

Este guia é um tutorial **básico** de instalação e uso do **Vagrant** em **Windows**, usando **VirtualBox** como provedor para os laboratorios fornecidos pela **SecDay**.  

Para mais informações **acesse nosso academy** em [https://academy.secday.com.br](https://academy.secday.com.br)

Ou nosso **canal no Youtube** em [https://www.youtube.com/@secday](https://www.youtube.com/@secday)

Inclui exemplos de:

- Instalar Vagrant  
- Conceitos básicos
- Subir uma VM **Linux** da SecDay
- Subir uma VM **Windows** da SecDay
- Subir uma VM **Kali** da SecDay  
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

1. **Windows 10 ou 11**, Linux **Ubuntu** ou **Debian** (64 bits), **macOS** (Testado apenas em processadores Intel) 
2. Acesso de administrador ou root para instalar programas
3. Espaço em disco de pelo menos 80GB
4. Processador minimo **core I5** ou **equivalentes AMD**
5. Pelo menos 8GB de RAM 
6. Suporte a virtualização ativado na BIOS/UEFI  
   - Normalmente chama-se **Intel VT-x**, **AMD-V** ou “Virtualization Technology”
7. **VirtualBox** Instalado
8. **macOS** gerenciador de pacotes [brew](https://brew.sh/) instalado
9. **Ubuntu/Debian**: wget instalado e pacotes apt atualizados `apt update -y && apt install wget -y`

---

## 3. Instalação

### 3.1. Instalar o Vagrant

1. Acesse o site oficial do [Vagrant](https://developer.hashicorp.com/vagrant/install).  
2. Baixe o instalador para **Windows** ou **Linux** (Dependendo do sistema Operaciona que utilize).  
3. **Windows**: Execute o instalador (`.msi`) e siga os passos padrão (ao final, você precisará reiniciar sua maquina).
4. **Linux** ou **macOS**: Executar a linha de comando fornecida pela documentação e aguardar instalação
5. Após instalar e reiniciar a maquina, abra o **PowerShell** ou **Prompt de Comando** e verifique:

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

## 5. Criando uma VM Linux da SecDay

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
  GRUPO_VBOX = "/SecDay"
  # Box base
  config.vm.box = "SecDay/ubuntu"
  config.vm.hostname = "ubuntu.local"
  config.vm.box_version = "1.0.0"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 22, host: 2221, protocol: "tcp", auto_correct: true
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.vm.provision "shell",
    inline: "apt-get update && apt-get install virtualbox-guest-additions-iso -y && reboot"

  # Nome da VM no VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.name = "Ubuntu-SecDay"
    vb.memory = "1024"
    vb.cpus = 1
    vb.customize ["modifyvm", :id, "--groups", "/secday"]
  end

end
```

### 5.4. Subir a VM Ubuntu da SecDay

Na pasta do projeto:

```powershell
vagrant up
```

O Vagrant irá:

1. Baixar a box (primeira vez), esse processo pode levar alguns minutos, a imagem `SecDay/ubuntu` tem em media 2GB de tamanho, então, dependendo da velocidade da sua internet, isso pode levar um tempo.
    
3. Criar a VM no VirtualBox
    
4. Iniciar a VM
    

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
  GRUPO_VBOX = "/SecDay"
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
    vb.customize ["modifyvm", :id, "--groups", "/secday"]
  end
end
```

### 6.4. Subir a VM Windows

Esse processo pode levar alguns minutos, a imagem `SecDay/windows2022` tem em media 9GB de tamanho, então, dependendo da velocidade da sua internet, isso pode levar um tempo.

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
## 7. Criando uma VM Kali da SecDay

### 7.1. Criar a pasta do projeto Windows

```powershell
mkdir C:\vagrant\kali
cd C:\vagrant\kali
```

### 7.2. Inicializar com a box Kali 

```powershell
vagrant init SecDay/kali
```

### 7.3. Exemplo de Vagrantfile para o Kali

```ruby
Vagrant.configure("2") do |config|
   GRUPO_VBOX = "/SecDay"
   config.vm.box      = "SecDay/kali"
   config.vm.hostname = "kali.local"
   config.vm.network "private_network", ip: "192.168.56.12"
   config.vm.network "forwarded_port", guest: 3389, host: 33892, protocol: "tcp", auto_correct: true
   config.ssh.username = "vagrant"
   config.ssh.password = "vagrant"
   config.vm.provision "shell",
     inline: "sudo apt install -y --reinstall virtualbox-guest-x11 && reboot"

   config.vm.provider "virtualbox" do |vb|
     vb.memory = 2048
     vb.cpus   = 1
     vb.name   = "kali-SecDay"
     vb.customize ["modifyvm", :id, "--groups", "/secday"]
   end
end
```

### 7.4. Subir a VM Kali

Esse processo pode levar alguns minutos, a imagem `SecDay/kali` tem em media 16GB de tamanho, então, dependendo da velocidade da sua internet, isso pode levar um tempo.

```powershell
vagrant up
```

Se tudo estiver configurado corretamente, o Vagrant irá subir a VM Kali no VirtualBox e se comunicar com ela via **SSH**.

### 7.5. Destruir a VM

```
vagrant destroy
```

Isso irá remover a VM criada e todos os seus arquivos. Uma mensagem de confirmação aparecerá na tela; selecione `y` para prosseguir com a exclusão.

### 7.6. Acessar a VM via SSH

```powershell
vagrant ssh
```

Isso já entra na VM como usuário padrão (`vagrant`), sem precisar configurar chave ou senha manualmente.

Para sair da VM:

```bash
exit
```

### 7.7 Acessando a VM Kali via Remote Desktop

A box Kali da SecDay vem com o RDP habilitado e você pode acessar pela porta de RDP:

No `Vagrantfile`, por exemplo:

```ruby
config.vm.network "forwarded_port", guest: 3389, host: 33892, auto_correct: true
```

Depois de subir a VM:

1. Abra **Conexão de Área de Trabalho Remota** no Windows (`mstsc`).
    
2. Conecte em: `localhost:33892`
    
3. Use o usuário/senha configurados na box (ex.: `vagrant` / `vagrant`).

---

## 8. Comandos úteis do Vagrant

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
## 9. Dicas finais

- Sempre execute os comandos do Vagrant na pasta onde está o `Vagrantfile`.
    
- Se der erro, rode:
    
    ```bash
    vagrant status
    vagrant up --debug
    ```
    
    para ver mais detalhes do problema.
    
- Verifique se o **VirtualBox** está atualizado e compatível com a versão do Vagrant.
    
- Evite abrir/alterar a VM diretamente pelo VirtualBox enquanto o Vagrant está gerenciando a máquina.

- Caso você precise alterar o diretório onde as imagens são armazenadas devido à falta de espaço em disco, abra o terminal PowerShell e execute o comando: `$env:VAGRANT_HOME = "D:\TMP\vagrantbox\"` (Altere o caminho para o diretório no disco onde você possui espaço disponível.)

- Você pode acessar diretamente a documentação para mais detalhes [aqui](https://developer.hashicorp.com/vagrant/docs)
    
