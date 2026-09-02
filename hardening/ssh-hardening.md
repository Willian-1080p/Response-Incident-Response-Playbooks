# SSH Hardening para Ubuntu/Linux

## Objetivo

Proteger o acesso SSH contra credenciais roubadas, força bruta e configurações inseguras sem causar perda de acesso administrativo.

> **Atenção:** mantenha uma sessão SSH ativa e acesso ao console. Abra uma segunda sessão para testar antes de encerrar a primeira. Não desative autenticação por senha até confirmar que a chave funciona.

## 1. Backup e Diagnóstico

```bash
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.bak-$(date +%F-%H%M)
sudo sshd -T
sudo ss -ltnp | grep ssh
```

Em versões recentes do Ubuntu, prefira um arquivo de inclusão como `/etc/ssh/sshd_config.d/99-hardening.conf`.

## 2. Chaves SSH

Na estação do administrador:

```bash
ssh-keygen -t ed25519 -a 100
ssh-copy-id usuario@servidor
```

Proteja a chave privada com frase secreta. Nunca copie a chave privada para o servidor. Confirme o login em uma nova sessão antes de prosseguir.

## 3. Configuração Recomendada

Crie `/etc/ssh/sshd_config.d/99-hardening.conf`:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
ClientAliveInterval 300
ClientAliveCountMax 2
LogLevel VERBOSE
```

Observações:

* Mantenha `PasswordAuthentication yes` até todas as chaves serem testadas.
* `AllowTcpForwarding no` e `AllowAgentForwarding no` podem quebrar túneis legítimos; habilite somente por exceção documentada.
* Não mude a porta SSH como controle principal. Isso reduz ruído, mas não substitui chaves, MFA, firewall e monitoramento.
* Use `AllowUsers usuario1 usuario2` ou `AllowGroups ssh-users` apenas após validar nomes e grupos.

## 4. Validar Antes de Aplicar

```bash
sudo sshd -t
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|kbdinteractiveauthentication|maxauthtries|allowtcpforwarding'
```

Se `sshd -t` mostrar erro, não recarregue o serviço. Corrija ou restaure o backup.

## 5. Aplicar com Segurança

```bash
sudo systemctl reload ssh
sudo systemctl status ssh --no-pager
```

Teste em outra janela:

```bash
ssh -o PreferredAuthentications=publickey usuario@servidor
```

Somente depois do teste, encerre a sessão original.

## 6. Restrição de Rede

Limite o acesso SSH à rede administrativa, VPN ou bastion:

```bash
sudo ufw allow from 192.0.2.0/24 to any port 22 proto tcp
sudo ufw status verbose
```

Substitua rede e porta pelos valores reais. Garanta a regra correta antes de remover uma regra ampla.

## 7. Fail2ban

Fail2ban complementa, mas não substitui, autenticação por chave e firewall:

```bash
sudo apt update
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Crie `/etc/fail2ban/jail.d/sshd.local`:

```ini
[sshd]
enabled = true
port = ssh
backend = systemd
maxretry = 5
findtime = 10m
bantime = 1h
```

Valide e inicie:

```bash
sudo fail2ban-client -t
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

Inclua redes administrativas confiáveis em `ignoreip` somente quando necessário e documentado.

## 8. MFA e Bastion

Para ambientes críticos, considere MFA via provedor de identidade, VPN ou bastion. Teste PAM e qualquer módulo MFA em homologação; uma configuração incorreta pode bloquear todos os administradores.

## 9. Monitoramento

```bash
sudo journalctl -u ssh --since today
sudo last -ai
sudo lastb -ai
sudo grep -R "Failed password\|Accepted" /var/log/auth.log*
```

Crie alertas para logins de origem incomum, acesso fora do horário, múltiplas falhas e mudanças em `authorized_keys` ou arquivos do SSH.

## 10. Permissões

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R "$USER":"$USER" ~/.ssh
```

Para outro usuário, execute de forma explícita com o proprietário correto. Verifique também se o diretório home não é gravável por terceiros.

## 11. Reversão

Se uma nova sessão falhar, use a sessão ainda aberta ou o console:

```bash
sudo rm /etc/ssh/sshd_config.d/99-hardening.conf
sudo sshd -t
sudo systemctl reload ssh
```

Se o arquivo principal foi alterado, restaure o backup correspondente. Registre a causa antes de tentar novamente.
