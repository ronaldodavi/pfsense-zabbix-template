# pfSense Modelo Zabbix

Este é um template ativo pfSense para Zabbix, baseado no Agente Padrão e um script php usando a biblioteca de funções pfSense para monitorar dados específicos.

Testado com pfSense 2.4.x, Zabbix 4.0, Zabbix 5.0

## O que faz

** PfSense Active Template **
 
 - Detecção e monitoramento de interface de rede com nomes atribuídos pelo usuário
 - Detecção e monitoramento do gateway (Status do gateway / RTT)
 - Detecção e monitoramento do servidor OpenVPN (status do servidor / status do túnel)
 - Detecção e monitoramento de clientes OpenVPN (status do cliente / status do túnel)
 - Monitoramento CARP (Estado CARP Global)
 - Descoberta e monitoramento de serviço básico (status do serviço)
 - Versão pfSense / atualização disponível
 - Pacotes de atualização disponíveis
 
** PfSense Active Template: OpenVPN Server User Auth **

 - Descoberta de clientes OpenVPN conectados a servidores OpenVPN no modo de autenticação do usuário
 - Monitoramento dos Parâmetros do Cliente (Bytes enviados / recebidos, Tempo de Conexão ...)

** Modelo ativo pfSense: IPsec **

 -Descoberta de túneis IPsec site a site
 - Monitorar o status do túnel (Fase 1 e Fase 2)
 
** PfSense Active Template: Speedtest **

 - Descoberta de interfaces WAN
 - Realizar testes de velocidade e coletar métricas


## Configuração

Primeiro copie o arquivo pfsense_zbx.php para sua caixa pfsense (por exemplo, para /root/scripts).

Em ** Diagnostics/Command Prompt **, insira esta linha:

```bash
curl --create-dirs -o /root/scripts/pfsense_zbx.php https://raw.githubusercontent.com/ronaldodavi/pfsense-zabbix-template/master/pfsense_zbx.php
```

Em seguida, instale o pacote "Zabbix Agent 4" em sua caixa pfSense


Em Recursos Avançados-> Parâmetros do Usuário

```bash
AllowRoot=1
UserParameter=pfsense.states.max,grep "limit states" /tmp/rules.limits | cut -f4 -d ' '
UserParameter=pfsense.states.current,grep "current entries" /tmp/pfctl_si_out | tr -s ' ' | cut -f4 -d ' '
UserParameter=pfsense.mbuf.current,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f1
UserParameter=pfsense.mbuf.cache,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f2
UserParameter=pfsense.mbuf.max,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f4
UserParameter=pfsense.discovery[*],/usr/local/bin/php /root/scripts/pfsense_zbx.php discovery $1
UserParameter=pfsense.value[*],/usr/local/bin/php /root/scripts/pfsense_zbx.php $1 $2 $3
```

_Por favor, note que a opção ** AllowRoot = 1 ** é necessária para executar corretamente as verificações do OpenVPN e outros._

Aumente também o valor de ** Tempo limite ** pelo menos para ** 5 **, caso contrário, algumas verificações falharão.

Em seguida, importe os modelos xml no Zabbix e adicione seus hosts pfSense.

Se você estiver executando uma configuração CARP redundante, deve ajustar a macro {$EXPECTED_CARP_STATUS} para um valor que representa o que é o status esperado do CARP na caixa monitorada.

Os valores possíveis são:

 - 0: Desativado
 - 1: Mestre
 - 2: Backup

Isso é útil ao monitorar serviços que poderiam permanecer interrompidos no membro reserva do CARP.

## Setup Speedtest
Para executar speedtests em interfaces WAN, você deve instalar o pacote speedtest.

Em ** Diagnostics/Command Prompt ** insira estes comandos:

```bash
pkg update && pkg install -y py38-speedtest-cli-2.1.3
```

Pode ser necessario procurar o pacote speedtest correto para isso.

Em ** Diagnostics/Command Prompt ** insira estes comandos:

```bash
pkg search speed
```

O pacote python do Speedtest pode estar quebrado no momento, então você pode precisar de uma etapa extra: baixe a versão mais recente do repositório github do autor do pacote.

```bash
curl -Lo /usr/local/lib/python3.8/site-packages/speedtest.py https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
```

Se houver erro no comando acima então devera ir.
Em ** Diagnostics/Edit file clicar em browser e seguir o caminho para alterar o comando acima para a pasta correta.

```bash
/usr/local/lib/
```bash

Para testar se o speedtest está instalado corretamente, você pode tentar:

```bash
/usr/local/bin/speedtest
```

Lembre-se de que você precisará instalar o pacote em * cada * atualização do pfSense.

O modelo Speedtest cria um cron job e verifica a entrada toda vez que o Zabbix solicita seus itens. Se você deseja desinstalar os cron jobs, basta executar em ** Diagnostics/Command Prompt **:

```bash
/url/local/bin/php/root/scripts/pfsense_zbx.php cron_cleanup
```

## Credits

[Ricardo Zabbix Template](https://github.com/rbicelli/pfsense-zabbix-template) Original Pfsense zabbix template

[Keenton Zabbix Template](https://github.com/keentonsas/zabbix-template-pfsense) for Zabbix Agent freeBSD part.
