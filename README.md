# PeguePagON | Sistema de Ponto de Venda Autônomo e Offline-First

Sistema comercial completo para autoatendimento e PDV autônomo, projetado para operar com tolerância a falhas de rede em micro e pequenos comércios (mercados de condomínio, padarias e conveniências).

A solução une aplicação móvel embarcada em terminais smart, sincronização em nuvem e painel administrativo web para controle de estoque e auditoria financeira.

---

## Dores do Mercado e Soluções Implementadas

### 1. Quedas frequentes de internet travando o atendimento
* **Dor:** Sistemas tradicionais baseados em navegadores ou APIs síncronas paralisam as vendas quando há oscilação de sinal.
* **Solução:** Arquitetura Offline-First real. As transações operam com armazenamento local de altíssimo desempenho via Hive. Quando a conectividade é restabelecida, filas de eventos sincronizam automaticamente com o Firestore na nuvem.

### 2. Complexidade de hardware e periféricos externos
* **Dor:** Necessidade de múltiplos cabos, impressoras fiscais separadas e pinpads lentos.
* **Solução:** Integração direta com terminais smart Android via Bridge nativa em Kotlin (MethodChannels), controlando a impressora térmica ESC/POS interna e leitores de código de barras no mesmo equipamento.

### 3. Fricção no pagamento
* **Dor:** Filas no caixa e demora para confirmar recebimentos.
* **Solução:** Geração dinâmica de QR Code PIX padrão EMV com validação automática de webhook e suporte a pagamentos integrados com split de transação.

### 4. Perda de dados e concorrência de estoque
* **Dor:** Estoques desincronizados entre o caixa da loja e a retaguarda.
* **Solução:** Modelo de dados reativo com reconciliação assíncrona, garantindo integridade de saldo mesmo com múltiplos caixas locais.

---

## Onde Essa Arquitetura se Aplica

Este projeto resolve gargalos em diversos nichos comerciais:
* **Micro markets e Honest Markets em condomínios:** Funcionamento 100% autônomo sem atendente.
* **Food trucks e quiosques itinerantes:** Operações em locais com sinal de celular instável.
* **Lojas de conveniência e panificação:** Vendas rápidas com emissão instantânea de comprovante.
* **Controle de eventos e feiras:** Múltiplos pontos de venda móveis sincronizando em tempo real com o backend central.

---

## Arquitetura de Software e Padrões

O projeto foi estruturado seguindo princípios de Clean Architecture, Domain-Driven Design (DDD) e alta coesão:

* **Padrão de Apresentação:** MVVM (Model-View-ViewModel) desacoplado da lógica de negócio.
* **Padrão Repository:** Abstração completa da fonte de dados, permitindo alternar de forma transparente entre o cache local (Hive) e a camada remota (Firebase/REST).
* **Camada Nativa (Platform Channels):** Código customizado em Kotlin para comunicação com o SDK do fabricante da maquininha (controle de hardware e impressora térmica).
* **Isolamento de Domínio:** Regras de cálculo de impostos, descontos e validações puramente declarativas, facilitando cobertura de testes automatizados.

---

## Tecnologias e Ferramentas

### Mobile & Aplicação Embarcada
* **Flutter & Dart:** Interface reativa multiplataforma.
* **Kotlin (Android Native):** Bridge nativa para comunicação com drivers de hardware.
* **Hive:** Banco NoSQL local ultrarrápido para persistência offline.
* **Firebase / Cloud Firestore:** Armazenamento distribuído e listeners de sincronização.

### Hardware & Periféricos
* **Terminais Android Smart POS:** Execução dedicada no modo Kiosk.
* **ESC/POS Nativo:** Impressão de cupons não fiscais e comprovantes em bobinas térmicas.
* **Câmeras / Scanners Integrados:** Leitura rápida de códigos EAN-13.

### Painel Web e Gestão
* **React, Vite e TypeScript:** Interface web administrativa para cadastro de produtos, gestão de estoque e relatórios financeiros.
* **Tailwind CSS:** Layout moderno e responsivo.

---

## Demonstração Visual

> *Insira aqui capturas de tela e GIFs da aplicação em execução:*

| Fluxo de Venda | Seleção de Produtos | Pagamento PIX EMV |
| :---: | :---: | :---: |
| ![Fluxo](assets/demo-fluxo.png) | ![Produtos](assets/demo-produtos.png) | ![PIX](assets/demo-pix.png) |

---

## Status do Código-Fonte

Por se tratar de um produto com aplicação comercial ativa desenvolvido pela **MicrotecON Sistemas Inteligentes**, o repositório principal com as chaves de integração e regras de negócio proprietárias permanece em ambiente privado sob licença fechada. 

Este repositório existe para fins de auditoria de arquitetura, documentação de padrões de engenharia de software e demonstração de capacidades técnicas.

---

## Autor

Desenvolvido por **Mateus Chagas**  
Engenheiro de Software | Fundador da MicrotecON Sistemas Inteligentes  
* LinkedIn: https://www.linkedin.com/in/mateuschagas/  
* GitHub: https://github.com/xmateuschagas
