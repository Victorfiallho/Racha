# 06 — Questões em aberto

Cada questão deve ser resolvida antes da implementação da parte afetada. Quando houver decisão, registre um ADR e atualize os requisitos.

| ID | Questão | Opções | Impacto |
|---|---|---|---|
| Q-01 | **Nome do produto e domínio.** | "Racha" é provisório | Branding, URLs nas peças (difíceis de trocar depois de impressas) |
| Q-02 | **Modelo de recebimento Pix:** quem é o recebedor e qual PSP usar? | (a) Credenciais Pix do próprio restaurante; (b) marketplace com split (Asaas, Mercado Pago, Efí, Pagar.me) | Regulatório, onboarding, taxa por transação. Ver ADR-0004 |
| Q-03 | **Como impedir que alguém de fora da mesa acesse a comanda?** | (a) Código curto exibido na peça e rotacionado por comanda; (b) código impresso na pré-conta; (c) sem código, apenas URL não enumerável | Fricção para o cliente vs. segurança |
| Q-04 | **A taxa de serviço é opcional para o cliente?** | Por restaurante; por participante | UX, cálculo |
| Q-05 | **O que fazer com o excedente** quando um item já pago é cancelado ou o preço cai? | (a) Estorno Pix automático; (b) crédito para a mesa; (c) o restaurante resolve manualmente | Financeiro, complexidade |
| Q-06 | **Modelo de cobrança da plataforma.** | Mensalidade por mesa; % por transação; híbrido | Viabilidade do negócio |
| Q-07 | **Stack.** | Candidatos: TypeScript (Node + PWA em React/Svelte), PostgreSQL com Supabase (RLS e Realtime nativos), hospedagem na Vercel ou similar | Velocidade de entrega |
| Q-08 | **Restaurantes do piloto e PDVs que eles usam.** | Levantar 3 a 5 restaurantes | Prioridade entre conectores e agente de impressão |
| Q-09 | **Gorjeta individual para o garçom** entra no MVP? | Sim / não | Rateio, repasse |
| Q-10 | **Licença do repositório.** | Privado/proprietário; MIT; AGPL | Estratégia de negócio |
