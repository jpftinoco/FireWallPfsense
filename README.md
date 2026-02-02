Implementação do Laboratório de Segurança
Este guia foi criado para orientar a configuração de um ambiente controlado de rede. O objetivo é isolar uma máquina de ataque (Kali) atrás de um firewall (pfSense), simulando uma rede corporativa real.

***Parte 1: Configurando o Firewall pfSense
O pfSense é um sistema operacional baseado em FreeBSD focado em firewall e roteamento.***

1. Criação da Máquina Virtual (VM)

Sistema Operacional: Ao criar a VM no VirtualBox, selecione FreeBSD (64-bit).

Por que 64-bit? Versões modernas do pfSense exigem arquitetura 64-bit. Se selecionar 32-bit, você enfrentará o erro "CPU doesn't support long mode", impedindo a inicialização do sistema.

2. Configuração de Rede (As "Portas" do Firewall)
Um firewall precisa de pelo menos duas interfaces para atuar como intermediário:

Adaptador 1 (WAN - Wide Area Network): Configure como Placa em modo Bridge. Isso permite que o pfSense "pegue" internet diretamente do seu roteador físico para atualizar pacotes e simular a conexão com o mundo exterior.

Adaptador 2 (LAN - Local Area Network): Configure como Rede Interna com o nome intnet. Esta será a rede privada onde o Kali Linux ficará "escondido". O tráfego nesta rede não sai para a internet sem passar pelas regras do pfSense.

3. Configuração de IPs via Console
No menu inicial do pfSense (tela preta), escolha a Opção 2 (Set interface(s) IP address).

Defina o endereço da LAN como 192.168.1.100 com máscara /24 (255.255.255.0).

Importante: Este endereço será o seu "Gateway Padrão", ou seja, o endereço que todas as máquinas internas usarão para sair da rede.

4. Criando a Política de Segurança (Firewall Rules)
Acesse o painel web pelo navegador do Kali em http://192.168.1.100.

Vá em Firewall > Rules > LAN.

Crie uma regra de Block para o protocolo IPv4 ICMP e selecione o subtipo Echo Request.


Atenção à Ordem: Esta regra deve estar no topo da lista. Firewalls leem regras de cima para baixo; se houver uma regra de "Permitir" antes dela, o bloqueio nunca será executado.

***Parte 2: Configurando o Kali Linux
O Kali Linux é a nossa plataforma de testes que ficará protegida (ou restrita) pelo firewall.***

1. Conexão Física Virtual
Nas configurações da VM Kali, vá em Rede e mude o Adaptador 1 para Rede Interna com o nome intnet.

Isso é o equivalente a conectar um cabo de rede do Kali diretamente na porta LAN do pfSense.

2. Configuração de Endereçamento Manual
Ambientes de laboratório isolados costumam ter falhas na entrega automática de IPs (DHCP). Por isso, configuramos manualmente:


IP: 192.168.1.101.


Gateway: 192.168.1.100 (O IP que demos ao pfSense).


Por que manual? Isso evita que a máquina perca a conexão ou use um IP de NAT padrão do VirtualBox, garantindo que ela pertença à rede 192.168.x.x controlada pelo firewall.

3. Teste de Validação (O "Ping da Morte")
No terminal do Kali, execute: ping -c 4 192.168.1.100.

O que deve acontecer: Você verá 100% de perda de pacotes.

Se o ping falhar, sua configuração foi um sucesso. Isso prova que o pfSense identificou o tráfego do Kali e aplicou a regra de bloqueio que você criou.

***Parte 3: Gestão de Identidade e Acesso (IAM)***
Além das regras de tráfego, o laboratório explorou o controle de quem pode gerenciar a infraestrutura.

1. Criação de Usuários e Grupos
O que foi feito: No menu System > User Manager, foram criados usuários específicos para separar as contas administrativas da conta padrão admin.

Por que isso é importante? Segue o princípio do Privilégio Mínimo. Em um ambiente real, você não compartilha a senha do admin. Cada colaborador tem seu usuário para que as ações sejam auditáveis (saber quem alterou qual regra).

2. Controle de Permissões (Privilégios)
O Desafio: Configurar quais usuários podem acessar a interface gráfica (WebGUI) e quais têm permissão apenas para visualizar logs ou status.

Configuração: Atribuição de "Privileges" específicos. Foi configurada a permissão WebCfg - Dashboard (all) apenas para usuários autorizados, bloqueando o acesso de outros perfis à gerência do firewall.
