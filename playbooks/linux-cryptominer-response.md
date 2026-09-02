# Linux Cryptominer Response Playbook

## Objetivo

Responder à mineração não autorizada em servidores Linux, identificar o vetor inicial e remover persistência sem ignorar comprometimentos mais amplos.

## Severidade

* **Alta:** minerador confirmado.
* **Crítica:** root, cloud credentials, propagação, botnet, exfiltração ou custos relevantes.

## Gatilhos

* CPU/GPU elevada, load anormal ou consumo de energia.
* Processo desconhecido, nome disfarçado ou executável em diretório temporário.
* Conexões persistentes a pools, Stratum ou domínios recém-criados.
* Cron/systemd que restaura o processo após encerramento.

## Ações Imediatas

Não encerre o processo antes de coletar PID, executável, linha de comando, pai, conexões e persistência. Restrinja saída de rede e isole o host se houver indícios de propagação.

## Evidências

```bash
date -u
uptime
ps aux --sort=-%cpu
sudo pstree -ap
sudo ss -tpna
sudo lsof -nP
sudo readlink -f /proc/PID/exe
sudo tr '\0' ' ' < /proc/PID/cmdline
sudo find /tmp /var/tmp /dev/shm -type f -mtime -7 -ls
sudo systemctl list-timers --all
```

Copie o binário sem executá-lo, calcule SHA-256 e preserve logs, cron, systemd e histórico de implantação.

## Investigação

Determine usuário, processo pai, início, pool/wallet, arquivos baixados e privilégios. Procure:

* exploração de aplicação, SSH fraco, credencial cloud ou container exposto;
* cron, systemd, shell profiles, chaves SSH e contas novas;
* ferramentas de varredura, roubo de credenciais e movimento lateral;
* o mesmo hash, wallet, domínio e IP em outros ativos.

Não trate mineração como incidente apenas de desempenho; frequentemente é consequência de acesso indevido.

## Contenção

* Bloqueie comunicação com pools nos controles de rede.
* Suspenda ou finalize o processo após coleta.
* Bloqueie conta comprometida e revogue sessões.
* Rotacione chaves, tokens e credenciais cloud a partir de sistema confiável.
* Isole nós/containers relacionados.

## Erradicação

Remova executável e persistência após preservação. Corrija vulnerabilidade ou credencial inicial, atualize aplicações e restrinja saída de rede. Se houve root ou integridade incerta, reconstrua o host.

## Recuperação

Restaure em imagem confiável, aplique hardening, limites de recursos e monitoramento de CPU/rede. Valide faturas cloud, recursos desconhecidos e IAM. Monitore por pelo menos 14 dias.

## Critérios de Encerramento

Vetor corrigido, persistência removida, credenciais revogadas, indicadores pesquisados, custos avaliados e host reconstruído ou integridade validada.
