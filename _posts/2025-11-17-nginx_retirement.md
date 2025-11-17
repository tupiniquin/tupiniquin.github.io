---
layout: post
title: Ingress Controller NGINX em Retirment
subtitle: O fim de uma era - o controlador de Ingress NGINX entra em retirement — e o que isso significa para você
gh-badge: [star, fork, follow]
tags: [NGINX, Kubernetes]
comments: true
image: https://cdn.prod.website-files.com/681e366f54a6e3ce87159ca4/6877c7726ae18757cf70352a_Kubernetes_Nginx_1.jpeg
---

![NGINX](https://cdn.prod.website-files.com/681e366f54a6e3ce87159ca4/6877c7726ae18757cf70352a_Kubernetes_Nginx_1.jpeg)

# Introdução

Nos últimos anos, o controlador Ingress‑NGINX tornou-se praticamente o padrão de fato para expor workloads HTTP/HTTPS dentro de clusters Kubernetes, especialmente em ambientes cloud-agnósticos ou híbridos. Agora, a comunidade do Kubernetes anuncia o seu **retirement**, com data-limite de manutenção em **março de 2026**. :contentReference[oaicite:2]{index=2}  
Este artigo examina o que está mudando, por que isso importa para infraestruturas de plataforma que você, como DevOps Engineer/Platform Engineer, provavelmente gerencia, e qual deve ser o plano de migração — com ênfase na adoção do Gateway API, o sucessor recomendado.

# O que está acontecendo?

A publicação oficial no blog do Kubernetes informa que:  
- O projeto Ingress-NGINX está em modo de “best-effort maintenance” até **março de 2026**. :contentReference[oaicite:4]{index=4}  
- Após essa data, **não haverá mais releases, correções de bugs nem atualizações de segurança** para esse controlador. :contentReference[oaicite:5]{index=5}  
- Os artefatos (imagens, Helm charts, chart repos, etc.) permanecerão disponíveis para referência, mas o repositório passará a read-only. :contentReference[oaicite:6]{index=6}  
- O time de manutenção da comunidade não conseguiu mais escalar suporte suficiente para o projeto, e as dívidas técnicas (e de segurança) acumuladas superam o limiar aceitável para continuar o projeto em modo ativo. :contentReference[oaicite:7]{index=7}  

Em resumo: se sua plataforma ainda usa Ingress-NGINX, é hora de se preparar para a transição. O controlador continuará funcionando — **mas sem garantias de segurança ou correções** — o que representa um risco de longo-prazo, principalmente em ambientes corporativos.

# Por que isso afeta sua plataforma?

Como DevOps Engineer / Platform Engineer — e considerando que você já trabalha com clusters Kubernetes, pipelines, GitOps, etc — aqui estão os impactos práticos que você deve considerar:

## Risco de segurança e compliance  
Sem correções de vulnerabilidade (CVE) após março-2026, qualquer falha descoberta passará a ser **não corrígivel** no contexto oficial. Dependendo da sua política de segurança, isso pode significar risco não mitigado ou falha em auditoria.

## Suporte e vida útil  
Mesmo que o controlador continue funcional, a falta de manutenção significa que novas versões de Kubernetes, mudanças de API, ou requisitos de infraestrutura podem quebrar ou deixar de funcionar corretamente com ele.

## End-of-life = dívida técnica  
Manter Ingress-NGINX além do suporte oficial representa acumular dívida técnica. Novas funcionalidades (por exemplo, suporte mais rico a roteamento, ambientes multi-tenant, políticas de tráfego avançadas) dificilmente serão atendidas. Além disso, migrar sob pressão no último momento implica mais custo e risco.

# Recomendação de migração: adote o Gateway API

A publicação oficial do Kubernetes recomenda explicitamente migrar para o Gateway API. :contentReference[oaicite:8]{index=8}

## O que é o Gateway API?  
O Gateway API é um conjunto de recursos do Kubernetes (CRDs) criado pelo grupo SIG‑Network para evolução da gestão de tráfego (ingress, load-balancing, roteamento L4/L7) em ambientes Kubernetes. :contentReference[oaicite:10]{index=10}  
Principais características:  
- Modelagem orientada a papéis (infra-estrutura, operador de cluster, desenvolvedor de aplicação). :contentReference[oaicite:11]{index=11}  
- Recursos distintos como GatewayClass, Gateway, HTTPRoute/GRPCRoute, que permitem separar responsabilidades. :contentReference[oaicite:12]{index=12}  
- Suporte para roteamento mais expressivo (cabeçalhos, weights, múltiplos protocolos, caminhos etc) sem depender de anotações específicas de controlador. :contentReference[oaicite:13]{index=13}  
- Portabilidade entre implementações, reduzindo dependência de controlador específico. :contentReference[oaicite:14]{index=14}  

## Por que migrar para ele agora 
- Você aproveita uma API moderna, suportada, com comunidade ativa e com roadmap — reduzindo risco de entrar em “legado” novamente.  
- Alinha sua plataforma com práticas de DevOps e Platform Engineering: abstração, controle de papéis, governança de tráfego, multi-tenant.  
- Migração antes de março 2026 dá uma janela confortável para planejamento, execução e mitigação de riscos — e evita fazer tudo na “corrida do fim”.  
- Permite padronização de roteamento e gateway para todo o seu ecossistema (apps, analistas, desenvolvedores).

# Plano de ação sugerido (como Platform Engineer)

Aqui vai um esqueleto de plano de ação que você pode adaptar ao seu contexto.  

1. **Inventário e análise de Ingress**  
   - Identifique todos os objetos de Ingress implementados por Ingress-NGINX nos seus clusters (por exemplo com `kubectl get ingress --all-namespaces -l app.kubernetes.io/name=ingress-nginx` ou selector equivalente).  
   - Verifique tags, anotações, snippets de NGINX usados — muitos controladores NGINX toleram “snippets” que são pouco portáveis ou seguros. A publicação menciona justamente que "snippets" foram uma das dívidas técnicas. :contentReference[oaicite:15]{index=15}  
   - Classifique os casos: simples (host+path) vs avançados (weights, mirroring, custom NGINX config).  

2. **Escolha piloto de Gateway API**  
   - Selecione um cluster (por ex. sandbox ou ambiente de desenvolvimento) para validar a migração para Gateway API.  
   - Instale CRDs do Gateway API e escolha implementação (por exemplo Envoy Gateway, Kong-Gateway, ou adequada ao seu ambiente).  
   - Crie recursos GatewayClass, Gateway e HTTPRoute para um serviço de teste — valide que roteamento e funcionalidades esperadas funcionam.

3. **Mapeamento e conversão**  
   - Para cada Ingress existente, mapeie para recursos equivalentes no Gateway API. Por exemplo:  
     * Ingress host+path → HTTPRoute host/path associado a Gateway.  
     * Listeners, TLS, backend weights → propriedades do Gateway/Route conforme Gateway API.  
   - Use guias de migração (há documentação no site oficial). :contentReference[oaicite:16]{index=16}  
   - Documente e adote padrões na sua plataforma (namespace padrão, Gateway compartilhado, RBAC, etc).

4. **Roll-out e testes**  
   - Após piloto OK, catalogue cronograma de migração para clusters de produção — alinhe com equipes de aplicação (já que você vai delegar para devs/analistas parte dos HTTPRoutes).  
   - Garanta rollback ou fallback à Ingress-NGINX enquanto estiver no período de suporte (até março 2026) para cada aplicação migrada.  
   - Monitore tráfego, métricas, logs e revise novos fluxos de tráfego.

5. **Descomissionamento controlado de Ingress-NGINX**  
   - Após migrar todas as cargas críticas, planeje a remoção do controlador Ingress-NGINX nos clusters onde não mais necessário, ou mantenha somente para cargas legadas com risco amortizado.  
   - Documente que após março 2026 o controlador não receberá patches de segurança — qualquer uso posterior será **por conta e risco**.

# Conclusão:

O retirement do Ingress-NGINX marca uma mudança significativa no ecossistema Kubernetes e reforça a necessidade de evoluir a arquitetura de tráfego nas plataformas modernas. Com o fim do suporte em março de 2026, torna-se essencial adotar soluções mais robustas, sustentáveis e alinhadas ao futuro da tecnologia — e o Gateway API surge exatamente com essa proposta.

Ao oferecer um modelo mais modular, seguro e flexível, o Gateway API se consolida como o novo padrão para expor aplicações e gerenciar tráfego em Kubernetes. Ele não apenas substitui o Ingress tradicional, mas eleva o nível de governança e observabilidade, trazendo previsibilidade e eficiência para equipes de desenvolvimento e operações.

Se você já utiliza Kubernetes em produção, este é o momento ideal para iniciar a transição. Migrar agora significa reduzir riscos, eliminar dívida técnica e preparar sua plataforma para os próximos anos — com uma API moderna, bem suportada e criada para acompanhar a evolução contínua do ecossistema cloud native.

**O futuro do tráfego no Kubernetes é o Gateway API. E quanto antes você fizer essa migração, mais preparada estará a sua plataforma.**

# Fontes utilizadas para elaboração do artigo:

**Documentação Oficial Kubernetes**  
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

**Blog Oficial Kubernetes**  
- https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/


\[ ESTE POST FOI FEITO PELO NOSSO TUTOR: [FELIPE CEZAR](https://www.linkedin.com/in/felipecezar88) ]

## Somos simples e raiz. Temos como objetivo, democratizar o acesso e compartilhar todo o tipo de conhecimento em tecnologia!!!

**< Tupiniquin Project >**
