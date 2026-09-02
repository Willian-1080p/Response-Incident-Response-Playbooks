# Linux Unauthorized User Response Playbook

## Objetivo

Responder à criação ou alteração não autorizada de contas, grupos, chaves SSH e privilégios em Ubuntu/Linux.

## Criticidade

* **Alta:** conta ou chave desconhecida sem uso confirmado.
* **Crítica:** login realizado, UID 0, sudo, acesso a dados ou movimento lateral.

## Gatilhos

* Novo usuário, grupo sudo ou UID 0.
* Chave desconhecida em `authorized_keys`.
* Shell ou senha alterados sem mudança aprovada.
* Login de conta desativada ou de serviço.

## Ações Imediatas

1. Registre host, conta, origem do alerta e horário UTC.
2. Preserve evidências antes de editar arquivos.
3. Se houver atividade, isole o host e bloqueie a conta de modo coordenado.
4. Não remova imediatamente a conta: isso altera timestamps e pode apagar contexto.

## Evidências

```bash
date -u
getent passwd
getent group sudo
sudo awk -F: '$3 == 0 {print $1}' /etc/passwd
sudo stat /etc/passwd /etc/shadow /etc/group /etc/gshadow
sudo last -Fai
sudo lastlog
sudo journalctl _COMM=useradd --since "30 days ago" --utc
sudo grep -E 'useradd|usermod|groupadd|passwd|Accepted' /var/log/auth.log*
sudo find /root /home -path '*/.ssh/authorized_keys' -type f -exec stat {} \;
```

Copie `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`, logs e chaves para local protegido; restrinja permissões e calcule hashes.

## Investigação

Determine:

* quem criou ou alterou a conta e por qual mecanismo;
* primeiro login, IP, autenticação e comandos;
* associação a sudo, grupos, containers e aplicações;
* chaves, tokens, cron, systemd e arquivos criados;
* outros hosts com a mesma conta ou chave.

Revise também cloud-init, ferramentas de configuração, IAM, scripts de implantação e tickets para descartar mudança legítima.

## Contenção

```bash
sudo usermod -L usuario
sudo usermod -s /usr/sbin/nologin usuario
sudo pkill -KILL -u usuario
```

Execute somente após preservar evidências e confirmar que não é conta de serviço crítica. Remova temporariamente do sudo, revogue chaves e sessões, restrinja rede e rotacione segredos acessíveis a partir de dispositivo confiável.

## Erradicação

* Remova conta, home e chaves somente após análise e aprovação.
* Corrija o vetor: credencial administrativa, automação, vulnerabilidade ou sudo indevido.
* Remova persistência associada.
* Se houve root, reconstrua o host a partir de imagem confiável.

## Recuperação e Encerramento

Valide contas, UID 0, grupos, sudoers, SSH, cron, systemd e aplicações. Monitore novas contas e logins por pelo menos 7 dias.

Encerre quando origem e escopo estiverem definidos, conta e sessões revogadas, segredos rotacionados, persistência removida e busca nos demais ativos concluída.
