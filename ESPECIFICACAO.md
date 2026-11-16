# Portal do Parceiro Parcer: especificação para desenvolvimento

Versão 1 · 16/11/2026 · Protótipo de referência: `app.html`, `index.html`, `formulario-saude.html`, `campanha-consorcio.html`

O protótipo guarda tudo no navegador (localStorage) só para validar a experiência. Este documento descreve o que o sistema real precisa ter.

## 1. Objetivo

Dar a cada corretora parceira um painel com login onde ela:
1. personaliza a própria marca (logo, cor, dados de contato);
2. publica uma página de captura com endereço próprio;
3. envia os formulários de cotação como link, com a marca dela;
4. recebe e organiza os leads.

A Parcer administra corretores, materiais, campanhas e formulários.

## 2. Perfis de acesso

| Perfil | Pode |
|---|---|
| Corretor (dono da corretora) | Editar a própria marca e página, ver e gerenciar só os próprios leads, copiar links |
| Usuário da corretora (opcional, v2) | Ver e atualizar leads da corretora |
| Admin Parcer | Cadastrar/desativar corretores, publicar materiais e campanhas, ver métricas agregadas |
| Cliente final (sem login) | Ver a página de captura e preencher formulários |

## 3. Telas (ver protótipo)

- **Login** (`app.html#login`): e-mail + senha, recuperação de senha por e-mail. Sem cadastro aberto: a Parcer convida o corretor.
- **Início** (`#painel`): leads do mês, visitas, link da página, checklist de primeiros passos.
- **Minha marca** (`#marca`): upload de logo, cor principal, nome, WhatsApp, e-mail, Instagram, SUSEP, cidade, endereço (slug). Prévia ao vivo.
- **Minha página** (`#pagina`): mensagem principal (3 opções aprovadas), texto de apoio, produtos exibidos, apresentação.
- **Formulários** (`#formularios`): 14 formulários; cada um com "ver como o cliente vê" e "copiar link".
- **Meus leads** (`#leads`): lista com filtro por status (Novo, Em contato, Fechado), detalhes, botão de WhatsApp.
- **Página pública** (`/c/{slug}`): hero, produtos, formulário rápido de interesse, quem somos, rodapé "com tecnologia PARCER".
- **Formulário público** (`/c/{slug}/{formulario}`): etapas, revisão, consentimento LGPD, envio.

## 4. Regras de personalização (obrigatórias)

- O corretor troca **logo, cor principal e textos**. Layout, fonte (Inter), espaçamentos e componentes são fixos.
- **Contraste automático:** se a cor escolhida tiver contraste menor que 3:1 com branco, o sistema escurece o tom até atingir 3:1 (função `theme()` em `app.html`). O tom escuro do hero é a cor misturada 42% com `#0B0D12`.
- Botão de WhatsApp sempre verde `#25D366`.
- Rodapé sempre com "com tecnologia PARCER" (logo oficial).
- Logo: PNG, JPG, SVG ou WEBP, até 3 MB; redimensionar para no máximo 480 px de largura. **Sanitizar SVG** (remover scripts) antes de servir.
- Slug: minúsculas, sem acento, só letras, números e hífen; único no sistema.

## 5. Dados

```
corretoras
  id, slug (único), nome, email, whatsapp, instagram, susep, cidade,
  cor, logo_url, head (0-2), texto_apoio, sobre, produtos (array),
  ativo, criado_em

usuarios
  id, corretora_id, email, senha_hash, papel (corretor|admin), ultimo_login

leads
  id, corretora_id, origem (pagina|formulario:<tipo>), produto,
  nome, telefone, email, respostas (json), status (novo|contato|fechado),
  consentimento_lgpd (bool + data/hora + IP), criado_em, atualizado_em

formularios
  id, tipo (saude, vida-individual, ...), titulo, schema (json com etapas e campos), ativo

eventos (métricas)
  id, corretora_id, tipo (visita|clique_whatsapp|envio), pagina, criado_em
```

**Formulários em JSON:** os 14 formulários devem virar `schema` (etapas → campos com tipo, rótulo, obrigatório, máscara, condição). Um único componente renderiza todos. O de Saúde no protótipo mostra os tipos necessários: texto, CPF (com validação), data, mês, telefone, e-mail, CEP (com busca ViaCEP), UF, escolha única em pílulas, select e campos condicionais.

## 6. Envio do lead

1. Salvar no banco (sempre, antes de qualquer outra ação).
2. Abrir o WhatsApp do corretor com a mensagem formatada (`wa.me/{numero}?text=...`).
3. Notificar o corretor por e-mail (v1) e push/WhatsApp API (v2, opcional).

## 7. LGPD e segurança

- Checkbox de consentimento obrigatório; guardar data, hora e IP do aceite.
- Política de privacidade da Parcer e da corretora linkada nos formulários.
- Cada corretora acessa **somente os próprios leads** (filtro por `corretora_id` no servidor, ou Row Level Security no Supabase).
- Senhas com hash (bcrypt/argon2); recuperação por link com expiração.
- Limite de envios por IP e proteção anti-spam (honeypot ou Turnstile) nos formulários públicos.
- HTTPS obrigatório. Backups diários.
- Retenção: definir prazo para apagar leads antigos e permitir exclusão a pedido do titular.

## 8. Stack sugerida

Duas opções:

**A. Dentro do Cockpit by Parcer**, que já tem login de corretor e hub de marketing. É o caminho preferido se o time do Cockpit tiver capacidade.

**B. Sistema novo e enxuto**
- Front: Next.js (ou HTML estático deste protótipo convertido em componentes)
- Login, banco e arquivos: Supabase (Auth + Postgres com RLS + Storage)
- Hospedagem: Vercel, com domínio `parceiro.parcerseguros.com.br` (painel) e `parcerseguros.com.br/c/{slug}` (páginas)
- E-mail transacional: Resend ou SendGrid

## 9. Fases

1. **MVP:** login, Minha marca, página de captura, formulário de Saúde, Meus leads, admin simples para cadastrar corretores.
2. **Formulários:** converter os outros 13 PDFs em schema.
3. **Posts com a logo do corretor:** gerador de artes (canvas) com as peças do calendário mensal + download PNG.
4. **Métricas e campanhas:** visitas, cliques, conversão por corretor; materiais e campanhas gerenciados pelo admin.

## 10. Identidade visual

Seguir `claude/parcer-contexto-marca.md`: Inter, paleta (Navy `#04163F`, Royal `#1D4FE8`, Sky `#7FB0FF`, Ice `#E8EEFC`, Soft `#F3F5FA`, Texto `#0A1B4A`, Muted `#5B6785`), títulos leves com palavra-chave em bold e ponto final, eyebrow em caixa alta espaçada. Estilos prontos em `css/parcer.css`.
