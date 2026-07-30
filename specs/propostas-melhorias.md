# 📋 Propostas de Melhorias e Adições

Documento de acompanhamento com sugestões de evolução para o projeto **Homelab Linux**, organizadas por prioridade. Cada item pode virar um módulo novo ou um ajuste em um módulo existente.

---

## 1. Consistência entre módulos

Os módulos `01-instalacao-debian`, `02-configuracao-ssh` e `03-instalando-nginx` usam um formato (tabelas, foco em saída de comandos) diferente dos módulos `04-samba-arquivos`, `05-configuracao-dns` e `06-firewall-nftables` (Objetivo / Tecnologias / Problema / Investigação / Solução / Conceitos aprendidos).

- [ ] Padronizar todos os módulos no formato usado em 04-06 (mais forte para portfólio, deixa claro o raciocínio de troubleshooting).
- [ ] Criar um `TEMPLATE.md` na raiz com essa estrutura, para os próximos módulos já nascerem padronizados.

---

## 2. Segurança

- [ ] **SSH**: migrar para autenticação por chave pública, desabilitar `PasswordAuthentication` e `PermitRootLogin` no `sshd_config`. Documentar o processo no módulo 02.
- [ ] **Fail2ban**: adicionar como novo módulo, protegendo o SSH contra brute-force.
- [ ] **Nginx**: habilitar HTTPS (certificado self-signed ou mini-CA local), já que existe DNS interno (`lab.local`).
- [ ] **Samba**: desabilitar NetBIOS (portas 137/138, legado) e forçar SMB2/3 no `smb.conf`.
- [ ] **nftables**: revisar a chain `output` (hoje `policy accept` totalmente aberta, com as regras de restrição comentadas no arquivo). Documentar se é decisão intencional ou pendência.
- [ ] **nftables**: confirmar e documentar a persistência do ruleset no boot (`systemctl enable nftables` + `/etc/nftables.conf`).
- [ ] **Atualizações automáticas**: configurar `unattended-upgrades` no Debian.

---

## 3. Reprodutibilidade

- [ ] Versionar os arquivos de configuração reais de cada serviço (`sshd_config`, `smb.conf`, `dnsmasq.conf`, server block do Nginx) dentro das respectivas pastas — hoje só `06-firewall-nftables/nftables.conf` está versionado como arquivo, os demais aparecem só como trecho no README.
- [ ] Avaliar um playbook **Ansible** que reproduza o setup inteiro (grande diferencial de portfólio, mostra IaC).

---

## 4. Navegação e portfólio

- [ ] Adicionar sumário com links para cada módulo no `README.md` da raiz.
- [ ] Adicionar diagrama simples de topologia de rede (servidor, roteador, cliente Windows).
- [ ] Adicionar `LICENSE` (ex.: MIT), comum em projetos educacionais no GitHub.

---

## 5. Roadmap de próximos módulos

- [ ] **Docker + Portainer** (já listado como "Em Breve" no README raiz).
- [ ] **Reverse proxy com TLS** (Nginx Proxy Manager ou Traefik) — evolução natural do módulo 03 + DNS.
- [ ] **Monitoramento** (Netdata é leve o suficiente para o hardware atual — Pentium Dual Core / 2GB RAM).
- [ ] **Backup de configs** (`etckeeper` ou script versionando `/etc`).
- [ ] **VPN** (WireGuard) para acesso remoto sem expor SSH diretamente.
