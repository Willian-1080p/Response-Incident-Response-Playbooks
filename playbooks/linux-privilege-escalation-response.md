# Linux Privilege Escalation Response Playbook

## Objetivo

Responder a suspeita de obtenção indevida de root ou de privilégios elevados em Ubuntu/Linux, preservando evidências e restaurando a confiança no sistema.

## Criticidade

Trate como **alta**. Eleve para **crítica** quando houver root confirmado, exploração remota, alteração de logs, persistência, acesso a segredos ou movimento lateral.

## Gatilhos

* comando sudo inesperado ou fora do horário;
* usuário adicionado ao grupo `sudo`;
* alteração não autorizada em `sudoers`, SUID/SGID ou capabilities;
* processo privilegiado iniciado por usuário comum;
* exploração conhecida no kernel ou serviço local;
* mudança de UID para 0, conta nova ou shell root.

## Ações Imediatas

1. Registre o alerta, horário em UTC, host, usuário e fonte.
2. Acione resposta a incidentes e proprietário do serviço.
3. Preserve uma sessão segura ou console fora de banda.
4. Isole o host conforme risco, mantendo coleta forense quando possível.
5. Não execute binários suspeitos nem remova artefatos antes de preservá-los.
6. Se root foi obtido, considere todo o host e os segredos acessíveis como comprometidos.

## Coleta Volátil e Evidências

```bash
date -u
who
w
ps auxf
sudo ss -tpna
sudo lsof -nP
sudo journalctl --since "24 hours ago" --utc
sudo journalctl _COMM=sudo --since "24 hours ago" --utc
sudo ausearch -m USER_CMD,USER_ROLE_CHANGE,ADD_USER,DEL_USER -ts recent 2>/dev/null
```

Copie logs e configurações para destino protegido:

```bash
sudo cp -a /var/log/auth.log* /caminho/evidencias/
sudo cp -a /etc/sudoers /etc/sudoers.d /caminho/evidencias/
sudo sha256sum /caminho/evidencias/* > /caminho/evidencias/SHA256SUMS
```

Mantenha cadeia de custódia. A coleta no próprio host pode ser enganada após root; quando possível, use imagem de disco, telemetria externa e ferramentas forenses confiáveis.

## Investigação

### Contas e Sudo

```bash
getent passwd
getent group sudo
sudo awk -F: '$3 == 0 {print $1}' /etc/passwd
sudo visudo -c
sudo find /etc/sudoers.d -type f -maxdepth 1 -ls
sudo grep -E 'sudo:|COMMAND=' /var/log/auth.log*
```

### SUID, SGID e Capabilities

```bash
sudo find / -xdev -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null
sudo getcap -r / 2>/dev/null
sudo find / -xdev -type f -mtime -7 -ls 2>/dev/null
```

Compare com uma linha de base ou pacote oficial. Não remova permissões apenas por aparecerem na lista.

### Persistência e Exploração

```bash
sudo systemctl list-unit-files --state=enabled
sudo systemctl list-timers --all
sudo find /etc/cron* /var/spool/cron -type f -ls 2>/dev/null
sudo dmesg -T | tail -200
uname -a
apt list --upgradable
```

Investigue:

* vetor inicial e vulnerabilidade explorada;
* primeiro e último evento;
* comandos executados como root;
* contas, chaves, serviços e módulos alterados;
* segredos, backups e outros hosts acessíveis;
* adulteração ou ausência de logs;
* indicadores em EDR, SIEM, IAM, rede e nuvem.

## Contenção

* Remova o host das redes não essenciais ou aplique ACL temporária.
* Bloqueie a conta afetada e revogue sessões.
* Revogue tokens, senhas e chaves acessíveis a partir de um sistema confiável.
* Desabilite temporariamente a função vulnerável se isso não destruir evidência.
* Bloqueie indicadores e procure-os em outros ativos.
* Preserve snapshots somente como evidência; não os trate automaticamente como backup limpo.

## Erradicação

* Aplique correção para a vulnerabilidade e ajuste permissões indevidas.
* Remova persistência somente após coleta.
* Corrija regras sudo, grupos, capabilities e arquivos alterados.
* Se root foi confirmado, reinstale a partir de mídia/imagem confiável. A limpeza manual não restabelece confiança suficiente.
* Restaure apenas dados e configurações verificados, nunca binários do host comprometido.

## Recuperação

1. Reconstrua o servidor com versão suportada e patches atuais.
2. Aplique hardening antes de conectá-lo à produção.
3. Use novos segredos e chaves; revogue os antigos.
4. Valide usuários, UID 0, sudo, SUID/SGID, capabilities, serviços, portas e aplicações.
5. Reintroduza o host por etapas.
6. Monitore sudo, execuções privilegiadas, rede e integridade por no mínimo 7 dias.

## Critérios de Encerramento

* Vetor e causa raiz identificados ou risco residual formalmente aceito.
* Escopo e linha do tempo documentados.
* Segredos e acessos afetados revogados.
* Host reconstruído ou integridade demonstrada por método aprovado.
* Busca por movimento lateral e impacto em dados concluída.
* Serviço, monitoramento e backups validados.
* Obrigações de comunicação e lições aprendidas concluídas.

## Melhorias Preventivas

* Menor privilégio e revisão de sudo.
* MFA e acesso administrativo via bastion/VPN.
* Auditoria de comandos privilegiados.
* Inventário de SUID/SGID e capabilities.
* AppArmor em modo enforce após testes.
* Patches com SLA por criticidade.
* Alertas para mudanças em usuários, sudoers, systemd, cron e logs.
