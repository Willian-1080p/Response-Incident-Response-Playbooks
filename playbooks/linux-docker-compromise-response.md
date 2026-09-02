# Linux Docker Compromise Response Playbook

## Objetivo

Responder a containers, imagens ou hosts Docker comprometidos, incluindo socket exposto, containers privilegiados e abuso de volumes.

## Criticidade

* **Alta:** container comprometido sem evidência de escape.
* **Crítica:** acesso ao socket Docker, container privilegiado, volume do host sensível, escape ou host comprometido.

## Gatilhos

* Processo ou conexão inesperada no container.
* Imagem desconhecida ou hash diferente do aprovado.
* Montagem de `/var/run/docker.sock`, `/` ou diretórios sensíveis.
* Container privilegiado, nova imagem, usuário ou tarefa não autorizada.

## Ações Imediatas

1. Registre horário UTC, host, container, imagem e orquestrador.
2. Restrinja rede do workload ou retire-o do balanceador.
3. Não remova container/imagem antes da coleta.
4. Se o socket ou host estiver exposto, isole o nó e trate-o como comprometido.

## Evidências

```bash
date -u
sudo docker ps --no-trunc
sudo docker images --digests --no-trunc
sudo docker inspect CONTAINER
sudo docker top CONTAINER
sudo docker logs --timestamps CONTAINER
sudo docker diff CONTAINER
sudo docker stats --no-stream
sudo ss -tpna
```

Exporte metadados e logs para destino protegido. Use `docker export` ou snapshot somente conforme procedimento forense; registre hashes. Colete também logs do daemon, registry, CI/CD, cloud e orquestrador.

## Investigação

Determine:

* imagem, digest, origem e assinatura;
* comando, usuário, capabilities, modo privilegiado e mounts;
* acesso ao socket Docker ou namespaces do host;
* segredos, tokens e credenciais montados;
* containers/imagens criados e registros acessados;
* possibilidade de escape e movimento lateral.

```bash
sudo docker inspect --format '{{.HostConfig.Privileged}} {{json .Mounts}}' CONTAINER
sudo journalctl -u docker --since "24 hours ago" --utc
sudo find /var/lib/docker -type f -mtime -7 -ls 2>/dev/null
```

Não altere diretamente `/var/lib/docker`.

## Contenção

Pare o workload após coleta: `sudo docker stop CONTAINER`. Revogue segredos, bloqueie a imagem/digest no registry e CI/CD, desabilite acesso remoto inseguro ao daemon e procure o mesmo digest em outros hosts.

## Erradicação

Reconstrua imagens a partir de fontes confiáveis, fixe digests, atualize base e dependências e remova privilégios/capabilities desnecessários. Se houver escape ou socket exposto, reconstrua o host.

## Recuperação

Implante novo container, não reinicie o comprometido. Use novos segredos, filesystem somente leitura quando possível, usuário não root, limites e política de rede. Monitore por pelo menos 14 dias.

## Critérios de Encerramento

Escopo do container e host definido, imagem bloqueada, segredos revogados, vetor corrigido, workloads reconstruídos e registry/CI/CD revisados.
