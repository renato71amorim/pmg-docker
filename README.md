# Proxmox Mail Gateway (PMG) via Docker Compose

[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![Proxmox](https://img.shields.io/badge/Proxmox-Mail%20Gateway-orange)](https://www.proxmox.com/en/proxmox-mail-gateway)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

Solução **Infraestrutura como Código (IaC)** para implantar o **Proxmox Mail Gateway** em containers Docker, de forma replicável, versionada e sincronizada com o GitHub.

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração](#-configuração)
- [Uso](#-uso)
- [Replicação em Outros Hosts](#-replicação-em-outros-hosts)
- [Segurança](#-segurança)
- [Limitações](#-limitações)
- [Manutenção](#-manutenção)
- [Licença](#-licença)

---

## 🎯 Sobre o Projeto

Este repositório fornece uma configuração **Docker Compose** pronta para implantar o **Proxmox Mail Gateway (PMG)** — uma solução open-source de gateway de e-mail com filtragem de spam, verificação de vírus e políticas de e-mail avançadas.

O objetivo é oferecer uma forma **simples, replicável e versionada** de subir o PMG em qualquer host Linux com Docker instalado, permitindo:

- **Centralizar** a filtragem de e-mails de múltiplas contas
- **Bloquear/excluir** mensagens indesejadas antes que cheguem à caixa de entrada
- **Versionar** toda a configuração via Git/GitHub
- **Replicar** o ambiente em diferentes servidores com poucos comandos

---

## 📁 Estrutura do Repositório

```
pmg-docker/
├── docker-compose.yml      # Definição dos serviços Docker
├── .env.example            # Template de variáveis de ambiente
├── .gitignore              # Arquivos ignorados pelo Git
├── README.md               # Este arquivo
├── config/                 # (ignorado) Configurações do PMG
├── data/                   # (ignorado) Dados persistentes
├── spool/                  # (ignorado) Spool de e-mails
└── postgres/               # (ignorado) Banco de dados PostgreSQL
```

> **Nota:** As pastas `config/`, `data/`, `spool/` e `postgres/` **não são versionadas** e armazenam todo o estado do PMG. Faça backup regular dessas pastas.

---

## ✅ Pré-requisitos

Antes de começar, certifique-se de ter instalado:

- **Docker** (versão 20.10 ou superior)
- **Docker Compose** (v2)
- **Git**
- Sistema operacional Linux (testado no **Linux Mint**)

Verifique com:

```bash
docker --version
docker compose version
git --version
```

---

## 🚀 Instalação

### 1. Clonar o repositório

```bash
git clone https://github.com/renato71amorim/pmg-docker.git
cd pmg-docker
```

### 2. Criar o arquivo de variáveis de ambiente

```bash
cp .env.example .env
```

### 3. Editar o `.env` com seus valores

```bash
nano .env
```

Defina:

| Variável | Descrição | Exemplo |
| :--- | :--- | :--- |
| `TZ` | Fuso horário | `America/Sao_Paulo` |
| `PMG_PASSWORD` | Senha do usuário `root` no PMG | `uma_senha_forte` |
| `PMG_DOMAIN` | FQDN do PMG | `pmg.seudominio.com.br` |

### 4. Subir o container

```bash
docker compose up -d
```

### 5. Verificar os logs

```bash
docker compose logs -f
```

Aguarde até que o PMG esteja pronto (pode levar alguns minutos na primeira execução).

---

## ⚙️ Configuração

Após a instalação, acesse o painel de administração do PMG:

- **URL:** `https://<IP-do-host>:8006`
- **Usuário:** `root`
- **Senha:** o valor definido em `PMG_PASSWORD`

### Configurações iniciais recomendadas

1. **Mail Proxy → Relay Domains:** adicione os domínios que serão protegidos.
2. **Mail Proxy → Relaying:** configure o host de destino (smarthost) para onde os e-mails limpos serão encaminhados.
3. **Spam Detector:** ative o filtro bayesiano, greylisting, SPF, DNSBLs.
4. **Mail Filter → Rules:** crie regras para **bloquear, descartar ou quarentenar** mensagens no nível SMTP.

---

## 🐳 Uso

### Comandos essenciais

```bash
# Subir o PMG em segundo plano
docker compose up -d

# Parar o PMG
docker compose down

# Reiniciar
docker compose restart

# Ver logs em tempo real
docker compose logs -f

# Atualizar a imagem para a versão mais recente
docker compose pull
docker compose up -d
```

### Backup dos dados

```bash
# Backup da pasta de dados (executar periodicamente)
tar -czvf pmg-backup-$(date +%Y%m%d).tar.gz config/ data/ spool/ postgres/
```

---

## 🔄 Replicação em Outros Hosts

Graças à abordagem **IaC**, replicar o ambiente em outro servidor é simples:

```bash
# 1. Clonar o repositório
git clone https://github.com/renato71amorim/pmg-docker.git
cd pmg-docker

# 2. Configurar variáveis para o novo host
cp .env.example .env
nano .env

# 3. Subir o container
docker compose up -d
```

### Atualizar configuração em todos os hosts

```bash
# No host de origem
git add .
git commit -m "Atualiza configuração X"
git push

# Em cada host de destino
git pull
docker compose up -d
```

---

## 🔒 Segurança

- **Nunca** faça commit do arquivo `.env` (ele já está no `.gitignore`).
- Use **senhas fortes** para `PMG_PASSWORD`.
- Considere usar **GitHub Secrets** ou um gerenciador de segredos (Vault, SOPS) em ambientes de produção.
- Restrinja o acesso à porta `8006` (painel) por firewall ou VPN.
- Mantenha a imagem Docker atualizada: `docker compose pull && docker compose up -d`.
- Faça **backup regular** das pastas `config/`, `data/`, `spool/` e `postgres/`.

---

## ⚠️ Limitações

- A imagem `dockurr/proxmox-mail` é **não oficial**. Para ambientes de produção críticos, prefira a instalação via **ISO** em VMs dedicadas.
- O **clustering nativo do PMG (HA Cluster)** não é suportado no modo Docker. Para alta disponibilidade real, use instalações bare-metal ou VMs.
- A replicação descrita aqui refere-se à **configuração via Git**, não à replicação de dados em tempo real entre nós.

---

## 🛠 Manutenção

### Atualizar a imagem

```bash
docker compose pull
docker compose up -d
```

### Limpar recursos não utilizados do Docker

```bash
docker system prune -a
```

### Verificar uso de recursos

```bash
docker stats pmg
```

---

## 📄 Licença

Este projeto está licenciado sob a **MIT License**. Consulte o arquivo `LICENSE` para mais detalhes.

O **Proxmox Mail Gateway** é um software da [Proxmox Server Solutions GmbH](https://www.proxmox.com/) e está sujeito à sua própria licença (AGPLv3).

---

## 🤝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir **issues** ou enviar **pull requests**.

---

## 📞 Contato

- **Autor:** Renato Amorim
- **Repositório:** [github.com/renato71amorim/pmg-docker](https://github.com/renato71amorim/pmg-docker)

---

**⭐ Se este projeto foi útil, considere dar uma estrela no GitHub!**
````

