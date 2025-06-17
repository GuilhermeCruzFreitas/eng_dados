# Streaming de video (netflix) vs data streaming

## Resumo

- **Streaming de vídeo** é sobre entrega de conteúdo multimídia ao consumidor final. É essencialmente transferência de arquivos otimizada para reprodução em tempo real, usando protocolos como HTTP/HTTPS sobre TCP.
- **Data streaming** é sobre o envio e processamento de eventos e dados em tempo real entre sistemas, usado para ETL, analytics e comunicação entre microserviços.

## Como a Netflix Entrega Vídeo

- A Netflix usa uma arquitetura de **Content Delivery Network (CDN)** distribuída:
	- **Armazenamento descentralizado**: Os vídeos ficam armazenados em servidores cache espalhados globalmente, incluindo dentro dos próprios provedores de internet (ISPs). Isso significa que quando você assiste algo popular, o vídeo pode estar literalmente a poucos quilômetros de você.
	- **Protocolo HTTP adaptativo**: Usa tecnologias como DASH (Dynamic Adaptive Streaming over HTTP) ou HLS (HTTP Live Streaming). O vídeo é dividido em pequenos segmentos de poucos segundos cada, com múltiplas qualidades disponíveis.
	- **Adaptação dinâmica**: Seu dispositivo constantemente monitora a velocidade da sua conexão e automaticamente escolhe a qualidade apropriada (240p, 480p, 1080p, 4K) para cada segmento. Se sua internet ficar lenta, a qualidade diminui; se melhorar, aumenta.
	- **Pré-carregamento inteligente**: O sistema baixa alguns segmentos à frente do que você está assistindo, criando um buffer. Isso previne pausas mesmo com pequenas variações na conectividade.
## Como Funciona na Prática

- **Segmentação**: O vídeo é pré-dividido em milhares de pequenos arquivos (2-10 segundos cada). Seu player baixa esses segmentos sequencialmente usando GETs individuais.
- **Manifesto**: Primeiro, seu dispositivo baixa um arquivo "manifesto" que lista todos os segmentos disponíveis e suas qualidades diferentes. É como um índice.
- **Downloads paralelos**: Enquanto você assiste o segmento 1, o player já está baixando os segmentos 2, 3 e 4 em background. Se sua internet acelerar, ele pode começar a baixar versões em qualidade maior dos próximos segmentos.
- **Adaptação contínua**: A cada poucos segundos, o algoritmo decide se deve baixar o próximo segmento em 720p, 1080p, ou 4K baseado na velocidade atual da sua conexão.
## Onde Entra o Data Streaming

Embora a entrega do vídeo não use data streaming, a Netflix usa massivamente essas tecnologias nos bastidores:
- **Analytics em tempo real**: Kafka e sistemas similares coletam bilhões de eventos sobre o que você assiste, quando pausa, pula, etc. Isso alimenta algoritmos de recomendação e decisões de negócio.
- **Monitoramento de qualidade**: Stream processing analisa métricas de performance de vídeo em tempo real para detectar problemas na rede de entrega.
- **Personalização dinâmica**: Dados sobre seu comportamento fluem em tempo real para ajustar recomendações, thumbnails personalizados e até mesmo decidir qual servidor deve entregar seu próximo vídeo.

A confusão entre streaming de serviços de assinatura e o streaming em engenharia acontece porque ambos lidam com "fluxos contínuos de dados", mas servem propósitos completamente diferentes. O streaming de vídeo é sobre entregar entretenimento; o data streaming é sobre processar informações operacionais que tornam essa entrega possível e otimizada. quando pensar em streaming para engenharia de dados, tenta focar principalmente na capacidade de entrega de dados que permita o consumo dos mesmo de maneira organizada e sequencial.