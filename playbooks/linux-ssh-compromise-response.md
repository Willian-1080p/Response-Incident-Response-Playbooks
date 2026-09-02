# Linux SSH Compromise Response Playbook

## Objetivo

Conter, investigar e recuperar um servidor Ubuntu/Linux quando há suspeita de acesso SSH não autorizado, preservando evidências e reduzindo o risco de persistência.

## Criticidade

* **Alta:** login suspeito confirmado, chave desconhecida ou conta administrativa afetada.
* **Crítica:** acesso root, persistência, exfiltração, movimento lateral ou alteração de logs.

## Gatilhos

* Login aceito de IP, horário ou país incomum.
* Nova chave em `authorized_keys`.
* Conta ou regra sudo criada sem autorização.
* Alertas de brute force seguidos de login bem-sucedido.
* Sessão, túnel SSH ou processo inesperado.

## Ações Imediatas

1. Acione o responsável pelo incidente e registre data/hora em UTC.
2. Não confronte o invasor nem execute ferramentas não confiáveis no host.
3. Preserve acesso por console ou canal fora de banda.
4. Isole o servidor na rede ou restrinja SSH a uma rede administrativa conhecida.
5. Se houver risco ativo, bloqueie contas e chaves comprometidas de forma coordenada.
6. Não desligue o host antes de avaliar a coleta de memória e processos voláteis.

> Se o servidor sustenta serviço crítico, coordene a contenção para evitar indisponibilidade maior. Não bloqueie um IP isolado como única contenção; o atacante pode mudar de origem.

## Preservação de Evidências

Registre cada comando, operador e horário. Prefira coletar para mídia ou servidor protegido:

```bash
date -u
who
w
last -Fai
sudo lastb -Fai
ps auxf
sudo ss -tpna
sudo journalctl -u ssh --since "24 hours ago" --utc
sudo cp -a /var/log/auth.log* /caminho/evidencias/
sudo find /home /root -path '*/.ssh/authorized_keys' -type f -exec stat {} \;
```

Calcule hashes depois da cópia:

```bash
sha256sum /caminho/evidencias/* > /caminho/evidencias/SHA256SUMS
```

Restrinja o diretório de evidências e mantenha cadeia de custódia. Evite modificar arquivos suspeitos antes de copiá-los.

## Investigação

### Autenticação e Escopo

```bash
sudo grep -E 'Accepted|Failed|Invalid user|session opened' /var/log/auth.log*
sudo journalctl _COMM=sshd --utc
getent passwd
getent group sudo
sudo find /root /home -path '*/.ssh/*' -type f -printf '%TY-%Tm-%Td %TH:%TM:%TS %u %m %p\n'
```

Determine:

* conta, método de autenticação, IP e primeiro horário comprometido;
* comandos executados e privilégios obtidos;
* hosts acessados e credenciais presentes;
* arquivos lidos, alterados ou transferidos;
* mecanismos de persistência;
* possível exfiltração e obrigações de notificação.

### Persistência

Revise sem remover inicialmente:

```bash
sudo systemctl list-unit-files --state=enabled
sudo systemctl list-timers --all
sudo crontab -l
sudo find /etc/cron* /var/spool/cron /tmp /var/tmp -type f -ls 2>/dev/null
sudo find /etc/systemd/system /usr/local/bin -type f -mtime -7 -ls
```

Ajuste o período à linha do tempo do incidente.

## Contenção

* Remova o host das redes não essenciais ou aplique ACL temporária.
* Desabilite a conta afetada: `sudo usermod -L usuario`.
* Revogue a chave comprometida e preserve uma cópia como evidência.
* Rotacione senhas, chaves, tokens e segredos acessíveis pelo host a partir de um dispositivo confiável.
* Revogue sessões no provedor de identidade/VPN.
* Bloqueie indicadores em firewall, bastion e SIEM, sem depender somente deles.
* Procure o mesmo usuário, chave e IP em outros servidores.

## Erradicação

* Remova contas, chaves, serviços, cron jobs e binários não autorizados após preservá-los.
* Corrija a causa raiz: credencial exposta, senha fraca, serviço vulnerável ou firewall aberto.
* Aplique atualizações e reforce SSH conforme `hardening/ssh-hardening.md`.
* Se houve root ou não é possível provar integridade, reinstale a partir de imagem confiável; não confie apenas em limpeza manual.

## Recuperação

1. Restaure dados validados em host reconstruído ou confiável.
2. Use novos segredos; não reutilize os presentes no sistema comprometido.
3. Valide usuários, sudo, portas, serviços, pacotes e hashes de aplicação.
4. Reabra acesso por etapas, limitado à rede administrativa.
5. Monitore autenticação, processos, rede e alterações por no mínimo 7 dias, ajustando ao risco.

## Critérios de Encerramento

* Causa raiz e período de comprometimento definidos.
* Todos os acessos e segredos afetados revogados.
* Persistência removida ou sistema reconstruído.
* Busca por impacto lateral concluída.
* Serviços validados pelo proprietário.
* Evidências, linha do tempo, decisões e notificações registradas.

## Lições Aprendidas

Implemente chaves com frase secreta, MFA, bastion/VPN, restrição de origem, inventário de chaves, alertas de login e revisão periódica de acessos.
