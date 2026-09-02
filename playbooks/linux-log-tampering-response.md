# Linux Log Tampering Response Playbook

## Objetivo

Responder à exclusão, alteração ou interrupção de logs do Linux, journald, rsyslog, auditd e aplicações.

## Criticidade

Trate como **alta**. Eleve para **crítica** quando a adulteração ocultar acesso root, exfiltração ou comprometer múltiplos ativos.

## Gatilhos

* Lacunas de tempo, truncamento ou rotação inesperada.
* Serviço de logs parado ou configuração alterada.
* Divergência entre logs locais, SIEM, rede e nuvem.
* Timestamps, permissões ou hashes incompatíveis.
* Limpeza de histórico do shell ou audit logs.

## Ações Imediatas

1. Registre horário UTC e fonte que detectou a lacuna.
2. Isole o host se houver atividade maliciosa.
3. Não reinicie serviços nem “corrija” logs antes de preservar evidências.
4. Proteja cópias remotas e limite acesso ao SIEM.
5. Considere ferramentas e timestamps locais não confiáveis após root.

## Evidências

```bash
date -u
timedatectl status
sudo systemctl status systemd-journald rsyslog auditd --no-pager
sudo journalctl --verify
sudo journalctl --disk-usage
sudo stat /var/log /var/log/auth.log /var/log/syslog 2>/dev/null
sudo find /var/log -type f -printf '%TY-%Tm-%Td %TH:%TM:%TS %u %m %s %p\n'
sudo journalctl --since "24 hours ago" --utc
```

Copie journal e logs preservando metadados; calcule hashes. Colete SIEM, firewall, proxy, IAM, cloud, EDR, banco e aplicações para reconstruir a linha do tempo.

## Investigação

Revise:

* comandos sudo e acessos antes da lacuna;
* mudanças em `/etc/systemd/journald.conf*`, `/etc/rsyslog*`, `/etc/audit*` e logrotate;
* unidades paradas, máscaras systemd e falhas de disco;
* NTP e alterações de relógio;
* uso de `journalctl --vacuum-*`, truncamento, remoção ou ferramentas de limpeza;
* outros sinais de root e persistência.

Diferencie adulteração de retenção normal, rotação, falta de espaço e falha operacional.

## Contenção e Erradicação

Bloqueie contas e sessões envolvidas, revogue segredos e restrinja o host. Restaure configuração a partir de fonte confiável somente após coleta. Se root ou binários de logging foram adulterados, reconstrua o host.

## Recuperação

Configure envio remoto protegido, retenção adequada, sincronização de tempo, monitoramento de integridade e alertas de parada/lacuna. Teste a chegada de eventos ponta a ponta.

## Critérios de Encerramento

Causa da lacuna definida, linha do tempo reconstruída na medida possível, acesso indevido contido, logging confiável restaurado e limitações de evidência documentadas.
