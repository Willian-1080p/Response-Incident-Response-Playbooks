# Linux Brute-Force Response Playbook

## Objetivo

Responder a tentativas repetidas de autenticação contra SSH e outros serviços Linux, distinguindo varredura automatizada de comprometimento real.

## Severidade

* **Baixa:** tentativas bloqueadas, sem conta válida ou sucesso.
* **Média:** alto volume, múltiplas origens ou degradação do serviço.
* **Alta:** tentativa seguida de autenticação aceita ou conta conhecida atacada.
* **Crítica:** acesso administrativo confirmado, persistência ou movimento lateral.

## Identificação

Indicadores:

* muitas mensagens `Failed password` ou `Invalid user`;
* picos de conexões na porta SSH;
* alertas do Fail2ban, firewall, IDS ou SIEM;
* login aceito logo após falhas;
* carga anormal no serviço de autenticação.

## Triagem Inicial

Registre o intervalo em UTC, serviço, IPs, usuários-alvo e volume:

```bash
date -u
sudo journalctl -u ssh --since "1 hour ago" --utc
sudo grep -E 'Failed password|Invalid user|Accepted' /var/log/auth.log
sudo ss -tn state established '( sport = :22 )'
sudo fail2ban-client status sshd
```

Não conclua que houve invasão apenas pelas falhas. Procure autenticações aceitas e atividade posterior.

## Preservação de Evidências

```bash
sudo cp -a /var/log/auth.log* /caminho/evidencias/
sudo journalctl -u ssh --since "24 hours ago" --utc > /caminho/evidencias/ssh-journal.txt
sha256sum /caminho/evidencias/* > /caminho/evidencias/SHA256SUMS
```

Documente origem, destino, porta, usuário, timestamps, regra de bloqueio e ações executadas.

## Análise

### Resumo de Origens e Usuários

```bash
sudo grep 'Failed password' /var/log/auth.log | awk '{for(i=1;i<=NF;i++) if($i=="from") print $(i+1)}' | sort | uniq -c | sort -nr
sudo grep 'Invalid user' /var/log/auth.log | awk '{print $(NF-5)}' | sort | uniq -c | sort -nr
sudo grep 'Accepted' /var/log/auth.log
last -Fai
```

Confirme manualmente a posição dos campos, pois o formato pode variar. No SIEM, correlacione falhas e sucessos pelo usuário, IP e janela de tempo.

Verifique:

* se a conta existe e possui sudo;
* se o IP pertence a VPN, scanner autorizado ou administrador;
* se houve login bem-sucedido após as tentativas;
* criação de usuários, chaves, cron jobs ou serviços;
* eventos semelhantes em outros ativos.

## Contenção

### Sem Login Bem-Sucedido

* Restrinja SSH a VPN, bastion ou redes administrativas.
* Aplique rate limiting/Fail2ban.
* Bloqueie origens de alto risco temporariamente, considerando falsos positivos.
* Preserve evidências e aumente o monitoramento.

### Com Login Bem-Sucedido ou Dúvida

* Trate como comprometimento e siga `linux-ssh-compromise-response.md`.
* Isole o host conforme criticidade.
* Bloqueie a conta e revogue sessões.
* Rotacione credenciais e chaves a partir de dispositivo confiável.
* Investigue movimento lateral.

## Proteções

Exemplo de Fail2ban em `/etc/fail2ban/jail.d/sshd.local`:

```ini
[sshd]
enabled = true
backend = systemd
maxretry = 5
findtime = 10m
bantime = 1h
```

```bash
sudo fail2ban-client -t
sudo systemctl reload fail2ban
sudo fail2ban-client status sshd
```

Combine com:

* chaves SSH e desativação de senha após teste;
* `PermitRootLogin no`;
* firewall com origens autorizadas;
* MFA, VPN ou bastion;
* alertas para falhas e sucessos anômalos.

Não dependa de troca de porta ou bloqueios permanentes de IP como defesa principal.

## Recuperação e Monitoramento

* Valide `sshd -t` antes de recarregar SSH.
* Teste acesso administrativo em segunda sessão.
* Confirme que usuários legítimos não foram bloqueados.
* Monitore por 72 horas ou mais conforme risco.
* Revise alertas em todos os hosts expostos.

## Critérios de Encerramento

* Não há evidência de autenticação não autorizada.
* Origens, usuários-alvo e período foram registrados.
* Controles de acesso foram corrigidos e testados.
* Indicadores foram pesquisados no ambiente.
* Se houve sucesso, o playbook de comprometimento foi concluído.
* Evidências e lições aprendidas estão anexadas ao incidente.
