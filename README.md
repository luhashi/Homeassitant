🧠 Home Assistant: O Cérebro do Meu Homelab
Este repositório documenta a configuração do meu Home Assistant, que atua como o orquestrador central e a interface de controle para toda a minha casa e meu homelab. Este projeto é uma peça fundamental na minha jornada de transição de carreira de videomaker para Engenheiro de IA e DevOps.

🎯 Filosofia
Vindo de um background de 10 anos no audiovisual, acredito em construir soluções que unem criatividade e eficiência técnica. Esta configuração do Home Assistant segue os mesmos princípios do meu homelab:

Prática: Resolve problemas do dia a dia, desde controlar luzes até gerenciar regras complexas de firewall.

Evolutiva: Está em constante aprimoramento, com novas automações e integrações sendo adicionadas regularmente.

Documentada: A documentação clara é a base de um projeto sustentável e escalável.

Segura: Opera dentro de uma arquitetura de rede Zero Trust, garantindo que a automação não comprometa a segurança.

🏛️ Contexto da Arquitetura
Meu Home Assistant não opera isoladamente. Ele está hospedado em um servidor Debian de baixo consumo e é parte integrante de um homelab mais amplo, gerenciado com Proxmox, Docker e Portainer. A segurança é um pilar central, com acesso externo e interno governado por princípios de Zero Trust.

O diagrama abaixo ilustra como o Home Assistant e outros serviços são acessados de forma segura:

    graph TD
    subgraph "External Access (Internet)"
        User_External["External User"]
    end

    subgraph "Cloudflare"
        CF_Proxy["Proxy (CDN/WAF)"]
        CF_Access["Access (SSO Login)"]
        CF_Tunnel["Tunnel"]
    end

    subgraph "Local Network (Homelab)"
        subgraph "VM: vm-docker-network"
            Cloudflared["cloudflared (Tunnel Agent)"]
            NPM["Nginx Proxy Manager"]
            AdGuard["AdGuard Home (Internal DNS)"]
        end

        subgraph "Internal Services"
            HA["Home Assistant"]
            OpenWebUI["OpenWebUI / LittleLLM"]
            UnifiCtrl["UniFi Controller"]
            OtherServices["...other services"]
        end

        User_Internal["Internal User / VPN Client"]
    end

    User_External -- "1. [https://service.example.com](https://service.example.com)" --> CF_Proxy
    CF_Proxy -- "2. Filters Threats (WAF)" --> CF_Access
    CF_Access -- "3. Forces SSO Login" --> CF_Tunnel
    CF_Tunnel -- "4. Secure Bridge (No Open Ports)" --> Cloudflared
    Cloudflared -- "5. Forwards ALL traffic to NPM" --> NPM

    User_Internal -- "[https://service.lab.example.com](https://service.lab.example.com)" --> AdGuard
    AdGuard -- "Replies with NPM's internal IP" --> User_Internal
    User_Internal -- "Accesses NPM directly" --> NPM

    NPM -- "Routes to correct service" --> HA
    NPM -- "Routes to correct service" --> OpenWebUI
    NPM -- "Routes to correct service" --> UnifiCtrl
    NPM -- "Routes to correct service" --> OtherServices

dashboards em destaque
Criei dashboards específicos para diferentes necessidades, focando em impacto visual e eficiência. Os dois principais são o Hub Central e a Central de Controle de Rede.

1. 🏠 Home - O Hub Central de Controle

Este é o dashboard principal para o dia a dia. Ele oferece controle intuitivo e rápido sobre todos os dispositivos IoT da casa, mídia e informações contextuais importantes.

Principais Funcionalidades:

Controle de Iluminação: Gerenciamento centralizado de luzes inteligentes, incluindo produtos comerciais (Yeelight, Philips Hue) e projetos DIY como uma fita de LED endereçável (WS2812B) controlada por um ESP32 com firmware WLED. A automação é enriquecida por uma rede Zigbee dedicada, onde sensores de presença e interruptores sem fio controlam luzes e acionam cenas com baixa latência.

Gerenciamento de Mídia: Controle unificado de players como Amazon Echo, Google Home e Spotify, além do controle de energia da TV Samsung.

Monitoramento de Dispositivos Pessoais: Cards que exibem o nível de bateria do iPhone, Apple Watch e MacBook.

Informações Ambientais: Exibição da previsão do tempo e dos horários do nascer e do pôr do sol.

2. 🌐 Network & Security - A Central de Controle de Rede

Esta é uma Central de Operações de Rede (NOC) construída inteiramente no Home Assistant, que proporciona visibilidade e controle granular sobre toda a infraestrutura de rede UniFi e serviços de segurança como o AdGuard.

Principais Funcionalidades:

Monitoramento de Infraestrutura UniFi: Status em tempo real, uptime, uso de CPU e memória do Gateway Lite e do Access Point U7 Lite, com botões para reinicialização remota.

Gerenciamento Dinâmico de Wi-Fi: Permite ativar ou desativar redes Wi-Fi segmentadas (Toca da Cacau, Toca da Sayuri, Toca do Mochi para convidados) e exibe um QR Code para acesso fácil.

Controle do AdGuard Home: Visão geral das estatísticas de bloqueio e switches para gerenciar os módulos de proteção.

Gerenciamento de Regras de Firewall: Switches que ativam/desativam dinamicamente regras no firewall da UniFi, como a que permite a comunicação do Home Assistant com a VLAN de IoT.

⚡ Automações em Destaque
O verdadeiro poder desta configuração está nas automações que conectam diferentes sistemas para criar uma experiência verdadeiramente inteligente e proativa.

🛒 Lista de Compras Inteligente

Uma lista de compras compartilhada entre mim e minha noiva permite que ambos adicionemos itens a qualquer momento. O Home Assistant possui as zonas dos principais supermercados que frequentamos configuradas. Ao detectar que cheguei em um desses locais, ele proativamente envia uma notificação para o meu celular com a lista de compras completa, garantindo que nada seja esquecido.

🎬 Modo Foco de Trabalho (Work Mode)

Com um único clique em um botão Zigbee no meu escritório, a cena "Work Mode" é acionada. Graças à integração com o HomeKit, esta mesma cena pode ser ativada por voz com a Siri ou diretamente do meu Apple Watch. A automação executa o seguinte:

As três lâmpadas do ambiente são ajustadas para uma iluminação ideal para trabalho.

Meu celular entra automaticamente no modo "Foco".

Minha playlist de trabalho do Spotify começa a tocar no alto-falante do escritório.

Bônus de Videomaker: Se a minha câmera principal for ligada (detectada pela rede), a automação pausa a música imediatamente para evitar problemas com a captação de áudio.

🚀 Alerta de Commit (DevOps)

Integrado ao meu fluxo de trabalho de desenvolvimento, o Home Assistant me notifica de forma visual e sonora sempre que um membro da equipe faz um commit em um dos repositórios que gerencio. Uma notificação é enviada para o meu celular e a luz da minha mesa pisca, servindo como um alerta periférico e imediato.

🏠 Modo Ausente Automático

Utilizando a geolocalização de nossos celulares como um sensor de presença combinado, o sistema detecta quando tanto eu quanto minha noiva estamos fora de casa. Nesse momento, ele executa uma rotina de "desligamento", garantindo que todas as luzes e dispositivos desnecessários sejam desligados para economizar energia.

☕ Bom Dia Personalizado (Em desenvolvimento)

Esta é a próxima automação a ser implementada. Ao ligar a máquina de café pela manhã (a primeira coisa que faço no dia), um sensor de consumo de energia detectará o pico e, combinado com o horário, entenderá que eu acordei. A Alexa, posicionada ao lado da máquina, me dará um "bom dia" personalizado com um resumo da minha agenda, a previsão do tempo e as principais notícias do dia, filtradas por hashtags de interesse.

🤖 AI Hub: Integrando Inteligência Artificial
Além do controle e monitoramento, transformei meu Home Assistant em um assistente de IA híbrido. Utilizo o LittleLLM como um proxy unificado para conectar o Home Assistant tanto a modelos de linguagem rodando localmente via Ollama quanto a provedores na nuvem como Gemini e Groq.

O fluxo de comunicação é o seguinte:
Home Assistant <--> Extended OpenAI <--> LittleLLM <--> (Ollama [Local] OU Gemini/Groq [Nuvem])

🛠️ Implementação Técnica
A mágica acontece através da combinação de integrações poderosas e da configuração declarativa dos dashboards com Lovelace UI.

Key Integrations

A funcionalidade deste projeto é possível graças a um conjunto de integrações essenciais, agrupadas por função:

Infraestrutura e Rede:

UniFi Network: Para controle e monitoramento profundo da infraestrutura de rede.

AdGuard Home: Para gerenciamento do bloqueador de DNS.

Proxmox VE: Para monitorar o status das VMs e do host.

Protocolos IoT:

Zigbee: Rede de baixa latência para sensores (presença, porta/janela) e interruptores, gerenciada via ZHA ou Zigbee2MQTT.

MQTT: Broker para comunicação desacoplada entre dispositivos IoT e integrações customizadas.

Ecossistema e Interoperabilidade:

HomeKit Bridge: Expõe todos os dispositivos relevantes (Zigbee, WLED, Wi-Fi) para o ecossistema Apple, permitindo controle via app Casa, Siri e Apple Watch.

Google Assistant / Alexa: Integração com assistentes de voz para comandos e automações.

Inteligência Artificial:

LittleLLM: Atua como um proxy unificado para LLMs.

Ollama: Permite a execução de modelos de linguagem locais.

Extended OpenAI Conversation: Integração que conecta o HA ao proxy LittleLLM.

Controle e Dados:

Iluminação Inteligente: Yeelight, Philips Hue e WLED para controle de produtos comerciais e fitas de LED customizadas com ESP32.

Mídia: Spotify, Broadlink, SmartThings.

Sensores: Home Assistant Mobile App para dados de dispositivos móveis.

Manutenção e Operações:

Google Drive Backup: Add-on que realiza backups automáticos e os envia para a nuvem, garantindo a recuperação de desastres.

RESTful Sensors: Sensores customizados para monitorar o status de outros serviços do homelab.

🛡️ Backup e Recuperação de Desastres

Para garantir a resiliência de todo o sistema, implementei uma estratégia de backup automatizada utilizando o add-on "Home Assistant Google Drive Backup". Esta é uma peça crítica da infraestrutura, protegendo contra falhas de hardware e corrupção de dados.

Automação Completa: Snapshots completos do Home Assistant são criados automaticamente em uma programação regular.

Armazenamento Off-site: Imediatamente após a criação, os snapshots são enviados para uma pasta dedicada no Google Drive, garantindo que os backups estejam seguros fora do hardware local.

Retenção Inteligente: O sistema gerencia automaticamente o número de backups a serem mantidos, tanto localmente quanto na nuvem, otimizando o espaço de armazenamento.

Monitoramento de Serviços Self-Hosted

Para garantir a saúde do ecossistema, utilizo a plataforma RESTful do Home Assistant para criar sensores binários que monitoram o status dos principais serviços do meu homelab. Isso permite criar alertas e ter uma visão rápida da saúde da infraestrutura diretamente nos dashboards.

<details>
<summary>Clique para expandir e ver o código configuration.yaml</summary>

# Exemplo de configuração no configuration.yaml para monitorar serviços
rest:
  # Nginx Proxy Manager - Verifica se a página de login está acessível
  - resource: [http://192.168.1.39:81/](http://192.168.1.39:81/)
    timeout: 10
    scan_interval: 60
    headers:
      User-Agent: "Home Assistant"
    binary_sensor:
      - name: "Nginx Proxy Manager Status"
        icon: mdi:gate-arrow-right
        value_template: "{{ 'html' in value }}"

  # Portainer - Verifica o endpoint de status da API
  - resource: [http://192.168.1.232:3010/api/status](http://192.168.1.232:3010/api/status)
    timeout: 10
    scan_interval: 60
    headers:
      User-Agent: "Home Assistant"
    binary_sensor:
      - name: "Portainer Status"
        icon: mdi:docker
        value_template: "{{ value_json.status == 'UP' }}"

</details>

Código dos Dashboards (Lovelace YAML)

O código YAML que estrutura os dashboards está disponível no repositório, utilizando vertical-stack, horizontal-stack e grid para uma organização eficiente.

🗺️ Próximos Passos
A automação é uma jornada, não um destino. Os próximos passos para este projeto incluem:

[x] Bom Dia Personalizado: Finalizar a implementação da automação de café.

[ ] Dashboards de Energia: Implementar monitoramento de consumo de energia para otimizar o uso e identificar gargalos.

[ ] Componentes Customizados: Desenvolver meus próprios componentes em Python para integrações que não existem nativamente, como monitoramento de progresso de render de vídeo.

Obrigado pela visita! Sinta-se à vontade para explorar o repositório.

