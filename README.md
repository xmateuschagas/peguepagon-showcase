# PeguePagON | PDV Autônomo e Offline-First

Sistema de ponto de venda para autoatendimento em pequenos comércios: padarias, mercadinhos de condomínio e conveniências. O cliente escolhe, pesa, paga no terminal e leva, sem atendente e sem depender de internet estável.

Produto da **MicrotecON Sistemas Inteligentes**, em produção desde julho de 2026, processando em média 180 vendas por dia em uma loja real.

> Este repositório é a vitrine técnica do produto. O código-fonte é proprietário e fica em repositório privado.

---

## Sumário

1. [Problemas de negócio resolvidos](#problemas-de-negócio-resolvidos)
2. [Visão geral da solução](#visão-geral-da-solução)
3. [Arquitetura](#arquitetura)
4. [Fluxo de uma venda](#fluxo-de-uma-venda)
5. [Tecnologias](#tecnologias)
6. [Decisões de engenharia](#decisões-de-engenharia)
7. [Demonstração](#demonstração)
8. [Autor](#autor)

---

## Problemas de negócio resolvidos

| Dor do comércio | Como o PeguePagON resolve |
|---|---|
| Internet cai e o caixa para | **Offline-first:** a venda é gravada no banco local (Hive) antes de qualquer chamada de rede. A sincronização com a nuvem acontece em segundo plano e reenvia pendências quando a conexão volta. |
| Loja sem atendente precisa vender sozinha | **Modo quiosque:** o app roda travado no terminal, com serviço de vigilância (watchdog) que reinicia a aplicação em caso de falha e envia telemetria e alertas. |
| Pagamento lento ou confirmado "de boca" | **Pagamento integrado:** cartão na maquininha e PIX com QR Code dinâmico, confirmados automaticamente por webhook. Nada é liberado sem confirmação do provedor. |
| Produtos pesados na balança | **Integração com balança:** leitura da etiqueta impressa pela balança (formato posicional de mercado) direto no leitor de código de barras. |
| Venda de itens restritos | **Verificação de idade** antes de liberar bebidas alcoólicas. |
| Dono longe da loja sem visão do caixa | **Painel web** com métricas em tempo real, cadastro de produtos e preços, e **relatório diário automático** por e-mail. |
| Estoque e preço diferentes entre loja e retaguarda | **Fonte de verdade única** no Firestore para catálogo, com cache local no terminal e merge controlado. |

---

## Visão geral da solução

```
┌────────────────────────────┐        ┌───────────────────────────┐
│  Terminal Android (loja)   │        │   Painel Web (dono)       │
│  Flutter + Hive            │        │   React + TypeScript      │
│  Leitor HID · Balança      │        │   Métricas · Catálogo     │
│  Impressora ESC/POS        │        │   Operadores · Relatórios │
└─────────────┬──────────────┘        └─────────────┬─────────────┘
              │ sync assíncrono                     │
              ▼                                     ▼
        ┌──────────────────────────────────────────────────┐
        │               Firebase (Firestore)               │
        │  produtos · vendas · operadores · telemetria     │
        └───────────────┬──────────────────────────────────┘
                        │
        ┌───────────────▼───────────────┐      ┌────────────────────┐
        │  Cloud Functions              │◄─────│ Provedor de        │
        │  webhook de pagamento         │      │ pagamento (cartão  │
        │  (HMAC + idempotência)        │      │ e PIX)             │
        └───────────────────────────────┘      └────────────────────┘
```

---

## Arquitetura

O app do terminal segue **Clean Architecture** com apresentação em **MVVM**, organizado em camadas com dependência sempre apontando para dentro:

```
presentation/   Telas (Views) + ViewModels (ChangeNotifier)
      │
      ▼
domain/         Entidades e regras de negócio puras (carrinho, promoções,
      │         caixa, sangria, fechamento, verificação de idade)
      ▼
data/           Repositórios: Hive (local) e Firestore (remoto)
      │
      ▼
platform/       Integrações nativas: pagamento, impressora, balança
                (MethodChannel / EventChannel em Kotlin)
```

**Princípios aplicados**

- **MVVM:** as telas só renderizam estado; toda decisão fica nos ViewModels.
- **Repository Pattern:** a camada de domínio não sabe se o dado veio do Hive ou do Firestore. Trocar a fonte de dados não mexe em regra de negócio.
- **Inversão de dependência nas integrações:** pagamento, impressora e balança são interfaces. Cada provedor tem sua implementação, e existe um mock para desenvolvimento e testes. Foi isso que permitiu trocar de provedor de pagamento sem reescrever o fluxo de venda.
- **Bridge nativa em Kotlin:** comandos pelo `MethodChannel` e status em tempo real pelo `EventChannel`, isolando o SDK do fabricante do restante do app.
- **Migração de dados sem perda:** adapters do Hive escritos à mão, com fallback de campos, para atualizar o app em produção sem corromper registros antigos.

---

## Fluxo de uma venda

```
1. Cliente monta o carrinho (toque, código de barras ou etiqueta da balança)
2. ViewModel valida regras (promoção, restrição de idade, estoque)
3. Venda gravada no Hive               → status: pendente (nunca se perde)
4. Cobrança criada no provedor         → cartão no terminal ou QR Code PIX
5. Webhook confirma o pagamento        → Cloud Function valida HMAC e
                                          descarta eventos duplicados
6. Terminal recebe a confirmação       → imprime comprovante ESC/POS
7. Sincronização com o Firestore       → imediata se online, fila se offline
```

---

## Tecnologias

**Aplicação do terminal**
- Flutter e Dart
- Kotlin (Android nativo) para bridges de hardware
- Hive (banco NoSQL local)
- Provider (estado e MVVM)
- Flutter Secure Storage (credenciais protegidas pelo Android Keystore)

**Backend e nuvem**
- Firebase: Cloud Firestore, Cloud Functions (Node.js), Hosting
- Webhook de pagamento com validação HMAC, idempotência e suíte de testes automatizados
- Relatório diário automático por e-mail

**Painel administrativo**
- React, Vite, TypeScript e Tailwind CSS

**Hardware e periféricos**
- Terminal Android smart POS com impressora térmica ESC/POS
- Leitor de código de barras HID
- Balança com etiqueta em formato posicional
- Etiquetas Code128

**Pagamentos**
- Cartão via maquininha integrada (modo PDV)
- PIX com QR Code dinâmico

---

## Decisões de engenharia

- **Por que offline-first?** Em loja sem atendente, uma venda perdida por queda de rede é prejuízo direto e sem ninguém para perceber. Gravar local primeiro elimina esse risco.
- **Por que webhook com idempotência?** Provedores reenviam notificações. Sem idempotência, um pagamento poderia ser contabilizado duas vezes.
- **Por que watchdog?** Um terminal travado em loja autônoma é loja fechada. O serviço em primeiro plano reinicia o app e manda telemetria, então o problema aparece no painel antes de virar reclamação.
- **Por que interfaces para hardware?** O mercado de maquininhas muda rápido. O produto já trocou de provedor de pagamento em produção sem tocar no fluxo de venda.

---

## Demonstração

Capturas de tela serão adicionadas na pasta [`assets/`](assets/).

| Venda | Pagamento PIX | Painel web |
|:---:|:---:|:---:|
| *em breve* | *em breve* | *em breve* |

---

## Status do código-fonte

O PeguePagON é um produto comercial ativo. O código, as credenciais de integração e as regras de negócio proprietárias ficam em repositório privado. Este repositório documenta a arquitetura e as decisões técnicas do projeto.

---

## Autor

**Mateus Chagas**, Engenheiro de Software e fundador da MicrotecON Sistemas Inteligentes
[LinkedIn](https://www.linkedin.com/in/mateusbchagas) · [GitHub](https://github.com/xmateuschagas)
