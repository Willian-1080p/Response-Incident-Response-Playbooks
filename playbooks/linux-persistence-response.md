# Linux Persistence Response Playbook

## Objetivo

Identificar e remover mecanismos de persistência em Ubuntu/Linux, incluindo cron, systemd, shell profiles, SSH, preload, timers e inicialização.

## Criticidade

Trate como **alta**; eleve para **crítica** se houver root, kernel, múltiplos hosts, exfiltração ou persistência que reaparece.

## Gatilhos

* Serviço, timer ou cron desconhecido.
* Alteração em `.bashrc`, `.profile`, `authorized_keys` ou sudoers.
* Processo que retorna após encerramento.
* Biblioteca preload, módulo, rc.local ou regra udev inesperada.

## Ações Imediatas

1. Registre horário UTC, host e artefato.
2. Isole o host conforme risco.
3. Não desabilite nem remova antes de preservar arquivo, metadados, logs e relações.
4. Se houver root, considere o host inteiro não confiável.

## Evidências

```bash
date -u
ps auxf
sudo ss -tpna
sudo systemctl list-unit-files --state=enabled
sudo systemctl list-timers --all
sudo crontab -l
sudo find /etc/cron* /var/spool/cron -type f -ls 2>/dev/null
sudo find /etc/systemd/system /usr/local/lib/systemd -type f -ls 2>/dev/null
sudo find /root /home -path '*/.ssh/authorized_keys' -type f -exec stat {} \;
sudo find /root /home -maxdepth 2 \( -name '.bashrc' -o -name '.profile' \) -type f -ls
sudo cat /etc/ld.so.preload 2>/dev/null
```

Inclua `/etc/rc.local`, init scripts, udev, PAM, sudoers, módulos, containers e cloud-init conforme o ambiente. Copie artefatos, calcule hashes e mantenha cadeia de custódia.

## Investigação

Para cada artefato, determine criador, timestamps, usuário, comando, binário, processo pai e conexões. Correlacione com:

* logs de sudo, SSH, auditd, implantação e EDR;
* contas/chaves novas;
* arquivos em `/tmp`, `/var/tmp`, `/dev/shm`;
* packages, capabilities e SUID/SGID;
* indicadores iguais em outros hosts.

Compare com baseline, repositório de configuração e tickets para excluir automação legítima.

## Contenção

Bloqueie comunicação maliciosa, conta e sessões. Suspenda o processo quando necessário para preservar evidência. Desabilite o mecanismo somente após coleta:

```bash
sudo systemctl disable --now nome.service
```

Não execute comandos presentes no artefato e não use curingas em remoções.

## Erradicação

Remova serviço, timer, cron, chave, script e binário associados após aprovação. Corrija o vetor inicial e rotacione segredos. Se persistência teve root, kernel ou não é totalmente compreendida, reconstrua o host.

## Recuperação

Reinstale a partir de imagem confiável quando indicado, restaure somente dados verificados, aplique hardening e monitore criação de tarefas, serviços, chaves e processos por pelo menos 14 dias.

## Critérios de Encerramento

Todos os mecanismos mapeados, vetor corrigido, artefatos preservados e removidos, segredos revogados, indicadores pesquisados e integridade restaurada ou host reconstruído.
