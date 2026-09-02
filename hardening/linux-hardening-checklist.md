# Checklist de Hardening Ubuntu/Linux

Use esta lista após instalação, mudanças relevantes e revisões periódicas. Marque **N/A** somente com justificativa documentada.

## Preparação

- [ ] Proprietário técnico, criticidade e função do servidor registrados.
- [ ] Versão do sistema e inventário de software registrados.
- [ ] Portas, serviços e fluxos de rede necessários documentados.
- [ ] Backup recente e restauração testada.
- [ ] Janela de manutenção e plano de reversão aprovados.
- [ ] Acesso ao console disponível antes de alterar rede ou SSH.

## Atualizações

- [ ] Repositórios APT são oficiais ou formalmente aprovados.
- [ ] Atualizações de segurança estão instaladas.
- [ ] `unattended-upgrades` está configurado e monitorado.
- [ ] Reinicializações pendentes são planejadas.
- [ ] Sistema operacional ainda recebe suporte de segurança.

## Identidades e Privilégios

- [ ] Cada administrador usa uma conta individual.
- [ ] Contas inativas ou desconhecidas foram bloqueadas/investigadas.
- [ ] Somente contas autorizadas pertencem ao grupo `sudo`.
- [ ] Acesso direto de root por SSH está desativado.
- [ ] Regras em `/etc/sudoers.d/` foram revisadas com `visudo`.
- [ ] Exceções `NOPASSWD` têm justificativa, responsável e validade.
- [ ] Contas de serviço não possuem shell interativo sem necessidade.
- [ ] MFA está habilitado para acesso administrativo quando suportado.

## SSH

- [ ] Chaves Ed25519 ou equivalentes aprovadas estão em uso.
- [ ] Chaves privadas têm frase secreta e armazenamento protegido.
- [ ] Login por chave foi testado em uma segunda sessão.
- [ ] Autenticação por senha foi desativada, quando viável.
- [ ] `PermitRootLogin no` está efetivo.
- [ ] Usuários ou grupos permitidos estão restritos.
- [ ] Encaminhamento, túnel e X11 estão desativados se desnecessários.
- [ ] SSH está restrito por firewall, VPN ou bastion.
- [ ] Logs e alertas de autenticação estão ativos.

## Rede e Serviços

- [ ] Política padrão do firewall nega conexões de entrada.
- [ ] Cada porta liberada tem aplicação, origem e responsável definidos.
- [ ] Serviços e pacotes desnecessários foram removidos ou desativados.
- [ ] Serviços escutam apenas nos endereços necessários.
- [ ] DNS, NTP e proxy usam fontes autorizadas.
- [ ] Não há credenciais ou painéis administrativos expostos publicamente.
- [ ] Regras de firewall foram testadas sem encerrar a sessão de segurança.

## Sistema e Kernel

- [ ] ASLR está habilitado.
- [ ] Redirecionamentos ICMP não necessários estão desabilitados.
- [ ] Proteções de hardlink e symlink estão habilitadas.
- [ ] Ajustes `sysctl` foram testados para VPN, roteamento e aplicações.
- [ ] Partições sensíveis usam opções de montagem adequadas quando compatíveis.
- [ ] Arquivos graváveis por todos foram identificados e justificados.
- [ ] Binários SUID/SGID foram inventariados e revisados.

## Aplicações e Segredos

- [ ] Aplicações executam com usuário dedicado e menor privilégio.
- [ ] Segredos não estão em código, histórico do shell ou arquivos públicos.
- [ ] Permissões de arquivos de configuração e segredos estão restritas.
- [ ] TLS usa certificado válido e configuração atual.
- [ ] Dependências e imagens têm origem confiável e são atualizadas.
- [ ] Serviços possuem limites e isolamento via systemd/AppArmor quando aplicável.

## Logs e Detecção

- [ ] Horário do sistema está sincronizado.
- [ ] `journald`, `rsyslog` e/ou `auditd` estão ativos conforme a política.
- [ ] Logs críticos são enviados para armazenamento remoto ou SIEM.
- [ ] Retenção e proteção contra alteração atendem aos requisitos.
- [ ] Alertas cobrem login anômalo, sudo, mudanças de usuários e persistência.
- [ ] Procedimentos de triagem e contatos de incidente estão atualizados.

## Backups e Continuidade

- [ ] Backups são criptografados, versionados e isolados.
- [ ] Credenciais de backup são separadas das administrativas.
- [ ] RPO e RTO estão definidos.
- [ ] Restauração foi testada com resultado registrado.
- [ ] Há cópia protegida contra exclusão/alteração pelo servidor de origem.

## Validação Final

- [ ] `sshd -t` e a configuração efetiva do SSH foram validados.
- [ ] Firewall, portas e serviços ativos correspondem ao inventário.
- [ ] Não há unidades systemd em falha sem explicação.
- [ ] Aplicação e monitoramento passaram em testes funcionais.
- [ ] Acesso administrativo alternativo foi testado.
- [ ] Alterações, exceções e evidências foram registradas.
- [ ] Próxima data de revisão e responsáveis foram definidos.

## Registro da Revisão

| Campo | Valor |
|---|---|
| Servidor/ativo | |
| Ambiente | |
| Data | |
| Responsável | |
| Mudança ou ticket | |
| Resultado | |
| Exceções e validade | |
| Próxima revisão | |
