# Linux Suspicious Process Response Playbook

## Objetivo

Investigar processos Linux suspeitos, como reverse shells, binários desconhecidos, consumo anormal de CPU/RAM e conexões inesperadas.

## Severidade

* **Média:** anomalia sem evidência de execução maliciosa.
* **Alta:** binário ou conexão maliciosa confirmada.
* **Crítica:** root, persistência, exfiltração ou movimento lateral.

## Gatilhos

* CPU, memória ou I/O anormais.
* Processo executado de `/tmp`, `/dev/shm` ou diretório oculto.
* Processo com nome semelhante a serviço legítimo.
* Conexão externa, shell filho ou arquivo deletado ainda em execução.

## Ações Imediatas

Não mate o processo antes de coletar contexto, salvo risco operacional imediato. Isole o host ou restrinja saída de rede e registre horário UTC.

## Coleta Volátil

```bash
date -u
uptime
ps auxf
sudo pstree -ap
sudo ss -tpna
sudo lsof -nP
sudo lsof +L1
sudo cat /proc/PID/status
sudo tr '\0' ' ' < /proc/PID/cmdline
sudo readlink -f /proc/PID/exe
sudo ls -la /proc/PID/cwd /proc/PID/fd
```

Substitua `PID` explicitamente. Copie o executável e arquivos relacionados para área protegida sem executá-los, calcule SHA-256 e registre proprietário, permissões e timestamps.

## Investigação

```bash
sudo sha256sum /proc/PID/exe
sudo dpkg -S /caminho/do/binario
sudo journalctl _PID=PID --utc
sudo find /tmp /var/tmp /dev/shm -type f -mtime -7 -ls
sudo systemctl status nome-do-servico
```

Determine processo pai, usuário, início, argumentos, arquivos, conexões, pacote de origem e persistência. Compare hash em fontes de inteligência aprovadas sem enviar arquivos sensíveis. Procure o mesmo hash, domínio, IP e comando em outros hosts.

## Contenção

* Bloqueie comunicação maliciosa em controles de rede.
* Suspenda o processo com `sudo kill -STOP PID` quando a preservação for necessária e segura.
* Finalize com `sudo kill -TERM PID`, escalando para `-KILL` apenas se necessário.
* Bloqueie a conta comprometida e revogue segredos.
* Se houver root ou dúvida de integridade, isole e planeje reconstrução.

## Erradicação

Preserve e remova executável, serviço, cron ou script de inicialização. Corrija vulnerabilidade ou credencial inicial. Atualize o sistema. Não confie apenas em encerrar o processo.

## Recuperação

Valide processos, portas, serviços, usuários e integridade de pacotes. Reative por etapas e monitore CPU, rede, arquivos e recorrência por pelo menos 7 dias.

## Critérios de Encerramento

Processo identificado, vetor corrigido, persistência removida, indicadores pesquisados no ambiente, segredos revogados e integridade do host restaurada ou host reconstruído.
