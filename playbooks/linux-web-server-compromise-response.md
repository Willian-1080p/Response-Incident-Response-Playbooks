# Linux Web Server Compromise Response Playbook

## Objetivo

Responder ao comprometimento de Apache/Nginx e aplicações web, incluindo webshell, alteração de conteúdo, exploração e acesso a dados.

## Criticidade

Trate como **alta**; eleve para **crítica** se houver webshell ativo, root, dados sensíveis, credenciais expostas ou movimento lateral.

## Gatilhos

* Arquivo web alterado, redirecionamento ou defacement.
* Processo do servidor iniciando shell.
* Requisições anômalas, upload executável ou alerta WAF/EDR.
* Usuário da aplicação criando cron, serviço ou conexão externa.

## Ações Imediatas

1. Preserve logs e registre horário UTC.
2. Retire a instância do balanceador ou aplique página de manutenção, quando possível.
3. Restrinja saída de rede e acesso administrativo.
4. Não apague o webshell antes de copiá-lo e coletar contexto.
5. Preserve uma instância/snapshot para forense; não o use como único backup limpo.

## Evidências

```bash
date -u
ps auxf
sudo ss -tpna
sudo lsof -nP
sudo journalctl -u nginx -u apache2 --since "24 hours ago" --utc
sudo cp -a /var/log/nginx /var/log/apache2 /caminho/evidencias/ 2>/dev/null
sudo find /var/www -type f -mtime -7 -printf '%TY-%Tm-%Td %TH:%TM:%TS %u %m %p\n'
```

Inclua logs da aplicação, WAF, proxy, CDN, banco e autenticação. Calcule hashes e mantenha cadeia de custódia.

## Investigação

* Identifique primeira requisição maliciosa, URI, IP, sessão e vulnerabilidade.
* Compare o webroot com versão do repositório ou artefato assinado.
* Procure funções de execução, conteúdo ofuscado e uploads recentes sem executar arquivos.
* Revise processos filhos, cron, systemd, chaves SSH, usuários e conexões.
* Determine dados acessados e segredos presentes em configurações e variáveis.
* Busque indicadores em outras instâncias.

## Contenção

* Remova a instância comprometida da produção.
* Bloqueie indicadores e rota vulnerável no WAF/proxy como medida temporária.
* Revogue sessões, tokens, chaves, senhas de banco e segredos acessíveis.
* Preserve disponibilidade com instância limpa, não clonada do host suspeito.

## Erradicação

Reconstrua a partir de imagem e artefato confiáveis. Corrija a vulnerabilidade, atualize dependências, remova uploads executáveis e aplique menor privilégio. Se houve root, não faça apenas substituição do webroot.

## Recuperação

Implante em nova instância, valide hashes/configurações, execute testes e reative tráfego gradualmente. Monitore requisições, processos, saída de rede e integridade por pelo menos 14 dias.

## Critérios de Encerramento

Vetor corrigido, escopo e dados afetados avaliados, segredos rotacionados, ambiente reconstruído, indicadores pesquisados e obrigações de comunicação concluídas.
