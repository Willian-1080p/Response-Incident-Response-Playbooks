# Ubuntu/Linux Server Hardening

## Objetivo

Reduzir a superfície de ataque de servidores Ubuntu/Linux sem comprometer a disponibilidade. Adapte os comandos à versão, à função do servidor e à política da organização.

> **Atenção:** execute em uma janela de manutenção, mantenha uma sessão administrativa aberta e acesso ao console. Teste primeiro em homologação. Faça backup dos arquivos antes de alterá-los.

## 1. Preparação e Inventário

Registre versão, serviços, portas, usuários e estado atual:

```bash
cat /etc/os-release
uname -a
sudo ss -tulpn
sudo systemctl --type=service --state=running
sudo awk -F: '$3 == 0 {print $1}' /etc/passwd
sudo cp -a /etc /root/etc-backup-$(date +%F)
```

Documente aplicações que precisam receber conexões e de quais redes.

## 2. Atualizações de Segurança

```bash
sudo apt update
sudo apt upgrade
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Verifique se o serviço está ativo:

```bash
systemctl status unattended-upgrades --no-pager
apt list --upgradable
```

Reinicie apenas após avaliar serviços críticos e a necessidade indicada por `/var/run/reboot-required`.

## 3. Contas e Privilégios

* Use contas individuais; não compartilhe credenciais.
* Conceda `sudo` somente a quem necessita e revise periodicamente.
* Bloqueie contas obsoletas com `sudo usermod -L usuario`.
* Não altere contas de serviço sem confirmar a dependência da aplicação.
* Exija senhas fortes e, quando possível, MFA no provedor de acesso.

Auditoria:

```bash
getent group sudo
sudo passwd -S root
sudo find / -xdev -perm -4000 -type f 2>/dev/null
```

Edite regras de sudo somente com `sudo visudo`. Prefira arquivos específicos em `/etc/sudoers.d/` e evite `NOPASSWD` sem justificativa.

## 4. Serviços e Pacotes

Liste e remova somente o que foi validado como desnecessário:

```bash
sudo systemctl disable --now nome-do-servico
sudo apt purge nome-do-pacote
sudo apt autoremove
```

Não desative serviços remotamente sem confirmar que não sustentam rede, acesso ou aplicação.

## 5. Firewall com UFW

Antes de habilitar, permita a porta SSH realmente usada e as portas da aplicação:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.0.2.0/24 to any port 22 proto tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

Substitua a rede de exemplo pela rede administrativa real. Não copie `192.0.2.0/24` como regra de produção.

## 6. Proteções do Kernel e da Rede

Crie `/etc/sysctl.d/99-server-hardening.conf` somente após verificar compatibilidade:

```text
kernel.randomize_va_space = 2
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.tcp_syncookies = 1
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
```

Aplicação e validação:

```bash
sudo sysctl --system
sudo sysctl kernel.randomize_va_space net.ipv4.tcp_syncookies
```

`rp_filter` pode afetar roteamento assimétrico, VPNs e hosts multihomed. Se houver impacto, reverta a chave específica.

## 7. Permissões e Sistemas de Arquivos

```bash
sudo chown root:root /etc/passwd /etc/group /etc/shadow /etc/gshadow
sudo chmod 644 /etc/passwd /etc/group
sudo chmod 640 /etc/shadow /etc/gshadow
sudo find / -xdev -type f -perm -0002 -print 2>/dev/null
sudo find / -xdev -type d -perm -0002 ! -perm -1000 -print 2>/dev/null
```

Investigue os resultados antes de corrigir. Aplicações podem depender de permissões específicas. Considere `nodev`, `nosuid` e `noexec` para partições adequadas, testando antes de editar `/etc/fstab`.

## 8. Logs, Auditoria e Tempo

```bash
sudo apt install auditd
sudo systemctl enable --now auditd
sudo systemctl enable --now systemd-timesyncd
timedatectl status
sudo journalctl -p warning --since today
```

Encaminhe logs para um servidor remoto/SIEM quando possível. Defina retenção conforme requisitos legais e capacidade.

## 9. AppArmor

```bash
sudo apt install apparmor apparmor-utils
sudo systemctl enable --now apparmor
sudo aa-status
```

Não coloque perfis em modo enforce sem observar negações e testar a aplicação.

## 10. Backups e Recuperação

* Mantenha cópias criptografadas, versionadas e fora do servidor.
* Separe credenciais de backup das credenciais administrativas.
* Teste restauração regularmente.
* Documente RPO, RTO e responsáveis.

## 11. Validação

Após cada bloco:

```bash
sudo ss -tulpn
sudo ufw status verbose
sudo systemctl --failed
sudo journalctl -p err -b
sudo apt-get check
```

Valide acesso SSH em uma segunda sessão antes de encerrar a primeira e teste as funções da aplicação.

## 12. Reversão

Restaure o arquivo de configuração alterado a partir do backup, reaplique a configuração e reinicie somente o serviço afetado. Para UFW, remova a regra específica com `sudo ufw status numbered` e `sudo ufw delete NUMERO`. Registre toda exceção, motivo, proprietário e data de revisão.

## Frequência Recomendada

* Diariamente: alertas, falhas, atualizações críticas.
* Mensalmente: usuários, sudo, portas, serviços e patches.
* Trimestralmente: regras de firewall, AppArmor, restauração de backup e exceções.
* Após mudanças ou incidentes: nova revisão completa.
