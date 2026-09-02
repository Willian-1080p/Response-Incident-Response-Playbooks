# Linux Rootkit Response Playbook

## Objetivo

Responder a suspeitas de rootkit em Ubuntu/Linux, incluindo processos ocultos, módulos de kernel maliciosos e adulteração de ferramentas do sistema.

## Criticidade

Trate como **crítica** quando houver ocultação confirmada, módulo desconhecido, alteração do kernel ou acesso root.

## Gatilhos

* Diferenças entre processos, portas ou arquivos vistos por ferramentas distintas.
* Módulo de kernel desconhecido, falhas de integridade ou binários do sistema alterados.
* Alertas de EDR, `rkhunter` ou `chkrootkit`.
* Logs ausentes, hooks suspeitos, processos que reaparecem ou tráfego oculto.

> Alertas de scanners podem ser falsos positivos. Não execute ferramentas baixadas de fonte desconhecida e não tente “limpar” antes da coleta.

## Ações Imediatas

1. Registre horário UTC, host, alerta e operador.
2. Isole o host da rede, preservando acesso forense fora de banda.
3. Não reinicie: isso pode destruir evidências voláteis ou acionar comportamento adicional.
4. Considere o sistema local não confiável; prefira telemetria externa e ferramentas em mídia confiável.
5. Acione equipe forense e responsáveis pelo serviço.

## Preservação de Evidências

Quando aprovado, capture memória antes de desligar. Registre estado volátil:

```bash
date -u
uname -a
ps auxf
sudo ss -tpna
sudo lsof -nP
sudo lsmod
sudo dmesg -T
sudo journalctl --since "24 hours ago" --utc
```

Crie imagem forense do disco com ferramenta aprovada, calcule hashes e mantenha cadeia de custódia. Evite instalar pacotes no host suspeito.

## Investigação

Compare a imagem offline com pacotes oficiais:

```bash
sudo debsums -s
sudo dpkg -V
sudo find /boot /lib/modules /etc/systemd/system -type f -mtime -14 -ls
sudo modinfo nome_do_modulo
```

Execute `rkhunter` ou `chkrootkit` apenas como fonte adicional, preferencialmente contra imagem montada e com base atualizada. Um resultado limpo não prova ausência de rootkit.

Correlacione:

* kernel, módulos e parâmetros de inicialização;
* processos, sockets, cron, systemd e bibliotecas preload;
* integridade de `ps`, `ss`, `ls`, `sudo`, SSH e logs;
* acesso root, vetor inicial, movimento lateral e segredos expostos.

## Contenção e Erradicação

* Bloqueie indicadores na rede e procure-os em outros hosts.
* Revogue credenciais, chaves e tokens acessíveis a partir de sistema confiável.
* Não confie em remoção manual.
* Reconstrua o host com mídia e imagem conhecidas, firmware atualizado e pacotes verificados.
* Corrija o vetor inicial antes da reconexão.

## Recuperação

Restaure apenas dados validados; não restaure binários ou configurações suspeitas. Aplique hardening, novos segredos e monitoramento reforçado. Reintroduza o serviço por etapas e monitore por pelo menos 14 dias.

## Critérios de Encerramento

* Evidências e linha do tempo preservadas.
* Vetor e escopo investigados.
* Host reconstruído a partir de fonte confiável.
* Segredos revogados e movimento lateral pesquisado.
* Aplicação, logs e controles validados.
