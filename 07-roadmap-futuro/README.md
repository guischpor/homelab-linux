# 🗺️ Projeto 07 - Roadmap de Próximos Módulos

## Objetivo

Consolidar o planejamento dos próximos serviços a implementar no homelab, com o passo a passo previsto para cada um. Este módulo funciona como guia de implementação — conforme cada item for executado no servidor, a seção correspondente deve ser atualizada com os resultados reais (saídas de comando, prints, problemas encontrados), seguindo o mesmo padrão dos módulos 01-06.

## Status

| Item | Status |
| :--- | :---: |
| Docker + Portainer | 🔴 Planejado |
| Reverse Proxy com TLS | 🔴 Planejado |
| Monitoramento (Netdata) | 🔴 Planejado |
| Backup de configs | 🔴 Planejado |
| VPN (WireGuard) | 🔴 Planejado |

---

## 1. 🐳 Docker + Portainer

### Objetivo
Rodar serviços em containers, isolando aplicações e facilitando deploy/atualização.

### Tecnologias
- Docker Engine
- Docker Compose
- Portainer CE (interface web de gerenciamento)

### Plano de implementação
```bash
# Instalar dependências e adicionar repositório oficial do Docker
sudo apt update
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Instalar Docker Engine + Compose plugin
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

# Adicionar usuário ao grupo docker (evitar uso de sudo constante)
sudo usermod -aG docker levi

# Subir o Portainer como container
docker volume create portainer_data
docker run -d -p 9443:9443 --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### Pontos de atenção
- Hardware do lab tem só 2GB de RAM — atenção ao número/peso dos containers rodando simultaneamente.
- Liberar a porta do Portainer (9443) no firewall nftables (módulo 06), restrita à rede local.

### Conceitos a aprender
- Diferença entre imagem e container
- Volumes e persistência de dados
- Docker Compose (multi-container)

---

## 2. 🔀 Reverse Proxy com TLS

### Objetivo
Centralizar o acesso aos serviços web (Nginx, Portainer, futuros containers) atrás de um único ponto de entrada, com HTTPS.

### Tecnologias
- Nginx Proxy Manager (via Docker) **ou** Traefik
- Certificado self-signed / mini-CA local (já existe DNS interno via `lab.local`)

### Plano de implementação
- Subir o Nginx Proxy Manager como container (depende do módulo Docker acima).
- Criar subdomínios no dnsmasq (módulo 05) para cada serviço, ex.: `portainer.lab.local`, `files.lab.local`.
- Gerar certificado self-signed (ou CA própria) e configurar HTTPS em cada proxy host.
- Redirecionar HTTP → HTTPS.

### Conceitos a aprender
- Reverse proxy vs. proxy tradicional
- TLS termination
- Virtual hosts / SNI

---

## 3. 📊 Monitoramento (Netdata)

### Objetivo
Visibilidade em tempo real de CPU, RAM, disco e rede — importante considerando o hardware limitado do servidor (Pentium Dual Core / 2GB RAM).

### Tecnologias
- Netdata (leve, dashboard web nativo, sem necessidade de stack Prometheus+Grafana)

### Plano de implementação
```bash
# Instalação via script oficial (modo estático, sem precisar de compilação)
curl -Ss https://my-netdata.io/kickstart.sh | sh

# Dashboard acessível em:
# http://192.168.1.145:19999
```

### Pontos de atenção
- Avaliar impacto do próprio Netdata no consumo de RAM/CPU do servidor.
- Restringir acesso à porta 19999 apenas à rede local via nftables.

### Conceitos a aprender
- Métricas de sistema (load average, I/O wait, etc.)
- Alertas e thresholds

---

## 4. 💾 Backup de Configurações

### Objetivo
Versionar automaticamente os arquivos de configuração do sistema (`/etc`), permitindo rollback e histórico de mudanças.

### Tecnologias
- `etckeeper` (versiona `/etc` com git automaticamente)

### Plano de implementação
```bash
sudo apt install etckeeper -y
sudo etckeeper init
sudo etckeeper commit "Estado inicial do /etc"
```

- Avaliar rotina adicional de backup externo (ex.: `rsync` para outro disco/máquina) dos dados do Samba (`/home/levi/debian_share`).

### Conceitos a aprender
- Versionamento de configs de sistema
- Estratégias de backup (3-2-1)

---

## 5. 🔒 VPN (WireGuard)

### Objetivo
Permitir acesso remoto seguro ao homelab (fora da rede local) sem expor SSH/serviços diretamente à internet.

### Tecnologias
- WireGuard

### Plano de implementação
```bash
sudo apt install wireguard -y

# Gerar par de chaves do servidor
wg genkey | tee privatekey | wg pubkey > publickey
```

- Configurar interface `wg0` no servidor com IP de VPN dedicado (ex.: `10.10.0.1/24`).
- Gerar par de chaves para o cliente (notebook/celular) e trocar chaves públicas.
- Liberar a porta UDP do WireGuard no roteador (port forward) e no firewall nftables — única porta exposta à internet.

### Pontos de atenção
- Com a VPN ativa, o SSH (módulo 02) não precisa mais ficar acessível fora da rede local — reforça a postura de segurança levantada em [`specs/propostas-melhorias.md`](../specs/propostas-melhorias.md).

### Conceitos a aprender
- Criptografia de chave pública aplicada a VPN
- NAT traversal / port forwarding
