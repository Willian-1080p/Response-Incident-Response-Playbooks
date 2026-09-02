# Linux Firewall Compromise Response Playbook

## Objetivo

Responder a mudanças não autorizadas em UFW, nftables ou iptables que exponham serviços, bloqueiem defesa ou permitam tráfego malicioso.

## Criticidade

* **Alta:** regra não autorizada sem exploração confirmada.
* **Crítica:** serviço exposto explorado, tráfego de comando e controle ou perda de acesso administrativo.

## Gatilhos

* Porta liberada, regra removida ou política padrão alterada.
* Firewall desativado ou serviço mascarado.
* Redirecionamento/NAT desconhecido.
* Divergência entre configuração declarada e estado efetivo.

## Ações Imediatas

1. Registre horário UTC e preserve acesso por console.
2. Não aplique reset remoto do firewall: isso pode bloquear administradores e apagar evidência.
3. Capture regras e contadores antes de alterar.
4. Se houver tráfego ativo, contenha também no firewall externo/security group.

## Evidências

```bash
date -u
sudo ufw status verbose
sudo ufw status numbered
sudo nft list ruleset
sudo iptables-save
sudo ip6tables-save
sudo ss -tulpn
sudo journalctl -u ufw -u nftables --since "24 hours ago" --utc
sudo stat /etc/ufw /etc/nftables.conf
```

Salve saídas e cópias das configurações em área protegida com hashes. Colete também security groups, firewall de borda, cloud audit e automação.

## Investigação

Determine:

* regra, autor, horário e mecanismo de alteração;
* origem da regra: UFW, nftables, iptables, Docker, Kubernetes ou ferramenta de configuração;
* serviço exposto e conexões durante a janela;
* comandos sudo, mudanças em arquivos e tarefas de persistência;
* se houve exploração ou exfiltração.

Mudanças de Docker podem gerar regras automaticamente; valide antes de classificar como maliciosas.

## Contenção

A partir do console ou com regra administrativa garantida:

* bloqueie tráfego malicioso em controle externo;
* restaure apenas as regras mínimas aprovadas;
* desabilite a conta ou automação responsável;
* isole serviços possivelmente explorados;
* preserve a configuração suspeita antes de substituí-la.

Valide sintaxe de nftables com `sudo nft -c -f /etc/nftables.conf` antes de aplicar.

## Erradicação

Remova persistência e credenciais comprometidas, corrija permissões e reconcilie regras com configuração versionada. Se a exposição permitiu invasão, siga o playbook do serviço comprometido.

## Recuperação

Teste acesso administrativo em segunda sessão, portas esperadas de dentro e fora, persistência após reinício planejado e alertas de mudança. Monitore por pelo menos 7 dias.

## Critérios de Encerramento

Origem da mudança definida, regras aprovadas restauradas, exposição investigada, acessos revogados, automação corrigida e monitoramento de integridade ativo.
