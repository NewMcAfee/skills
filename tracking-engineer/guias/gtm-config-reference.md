# Guia de Configuração GTM — Referência Operacional
> Carregado sob demanda pelo Tracking Engineer nas Fases 2 e 4.

---

## Seção 1 — Variáveis GTM

### Nomenclatura padrão por grupo

| Grupo | Prefixo | Exemplo |
|---|---|---|
| Permanentes editáveis | `[EDIT] Perma |` | `[EDIT] Perma | GA4 ID` |
| DataLayer | `DLV |` | `DLV | form_id` |
| Derivadas (ambiente) | `Derived |` | `Derived | Page URL` |
| User Data (hash) | `UD |` | `UD | Email Hash` |
| Cookie / Storage | `CKV |` | `CKV | fbclid` |

---

### Grupo 1 — Permanentes Editáveis `[EDIT] Perma |`

Tipo GTM: **Constante**. O operador substitui os valores `X` pelos IDs reais do projeto.

| Nome da Variável | Valor a preencher |
|---|---|
| `[EDIT] Perma | GA4 ID` | `G-XXXXXXXX` |
| `[EDIT] Perma | Meta Pixel ID` | `XXXXXXXXXX` |
| `[EDIT] Perma | GAds Conversion ID` | `AW-XXXXXXXXX` |
| `[EDIT] Perma | Clarity ID` | `XXXXXXXXXX` |
| `[EDIT] Perma | GAds Label | Lead` | label de conversão Lead |
| `[EDIT] Perma | GAds Label | MQL` | label de conversão MQL |
| `[EDIT] Perma | GAds Label | NOICP` | label de conversão NOICP (se no scope) |

> Adicione apenas labels para eventos que estão no scope do projeto. Não crie labels especulativos.

---

### Grupo 2 — DataLayer `DLV |`

Tipo GTM: **Data Layer Variable**. O nome no campo "Data Layer Variable Name" deve ser idêntico ao do `dataLayer.push()` — case-sensitive.

**Identidade do evento:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | event_id` | `event_id` |
| `DLV | event_time` | `event_time` |

**Formulário:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | form_id` | `form_id` |
| `DLV | form_name` | `form_name` |
| `DLV | form_type` | `form_type` |

**Qualificação:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | lead_status` | `lead_status` |
| `DLV | qualification_rule` | `qualification_rule` |
| `DLV | qualification_reason` | `qualification_reason` |

**User data** (inclua somente se o handoff declara user data no evento):
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | user.email` | `user.email` |
| `DLV | user.phone` | `user.phone` |
| `DLV | user.first_name` | `user.first_name` |

**Atribuição:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | attribution.utm_source` | `attribution.utm_source` |
| `DLV | attribution.utm_medium` | `attribution.utm_medium` |
| `DLV | attribution.utm_campaign` | `attribution.utm_campaign` |
| `DLV | attribution.utm_content` | `attribution.utm_content` |
| `DLV | attribution.fbclid` | `attribution.fbclid` |
| `DLV | attribution.gclid` | `attribution.gclid` |

---

### Grupo 3 — Derivadas `Derived |`

Tipo GTM: **URL** ou **JavaScript Variable**

| Nome da Variável | Tipo | Componente |
|---|---|---|
| `Derived | Page URL` | URL | Full URL |
| `Derived | Page Path` | URL | Path |
| `Derived | Page Hostname` | URL | Hostname |
| `Derived | Referrer` | JS Variable | `document.referrer` |

---

### Grupo 4 — User Data Hasheado `UD |`

Tipo GTM: **Custom JavaScript Variable**

Necessário para: Meta CAPI (Advanced Matching), Google Ads Enhanced Conversions.

**Regras de normalização obrigatória antes do hash:**
- Email: converter para lowercase, remover espaços
- Phone: formato E.164 com código do país (`+5511999999999`)
- Executar SHA256 após normalização

```javascript
// UD | Email Hash
function() {
  var email = {{DLV | user.email}};
  if (!email) return undefined;
  email = email.toLowerCase().trim();
  // SHA256 via CryptoJS ou implementação nativa
  return CryptoJS.SHA256(email).toString();
}
```

> Se o projeto usa um template de hash já instalado no GTM (ex: template oficial do Google para Enhanced Conversions), use-o e referencie a variável gerada ao invés de criar Custom JS.

---

## Seção 2 — Configuração de Tags por Plataforma

### 2.1 GA4 — Event Tag

**Tipo GTM:** Google Analytics: GA4 Event (ou via Google tag se configuração unificada)

Para cada evento canônico, crie uma tag:

```
Tag: TAG | GA4 | [EventoCanônico]
Measurement ID: {{[EDIT] Perma | GA4 ID}}
Event Name: [EventoCanônico]   ← exato, case-sensitive
Parameters:
  event_id    → {{DLV | event_id}}
  form_id     → {{DLV | form_id}}         (quando aplicável)
  lead_status → {{DLV | lead_status}}     (quando aplicável)
  qualification_rule → {{DLV | qualification_rule}}  (MQL/NOICP)
  utm_source  → {{DLV | attribution.utm_source}}
  utm_campaign → {{DLV | attribution.utm_campaign}}
Trigger: TRG | Custom Event | [EventoCanônico]
```

> QA: valide no GA4 DebugView (ative via GTM Preview — o debug_mode é setado automaticamente).
> Se Internal Traffic filter estiver ativo, eventos do DebugView não aparecem nos relatórios padrão — isso é comportamento esperado.

---

### 2.2 Meta Pixel — Browser Tag

**Tipo GTM:** HTML Customizado

```html
<script>
!function(f,b,e,v,n,t,s){...}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', '{{[EDIT] Perma | Meta Pixel ID}}', {
  em: '{{UD | Email Hash}}',
  ph: '{{UD | Phone Hash}}'
});
</script>
```

**Tag de evento (Lead):**
```html
<script>
fbq('track', 'Lead', {
  eventID: '{{DLV | event_id}}'
});
</script>
```

**Tag de evento customizado (MQL):**
```html
<script>
fbq('trackCustom', 'MQL', {
  eventID: '{{DLV | event_id}}',
  lead_status: '{{DLV | lead_status}}'
});
</script>
```

> `eventID` é obrigatório para deduplicação com CAPI. Deve ser idêntico ao `event_id` enviado pelo server. Ausência de `eventID` é causa de 80% dos erros de deduplicação Meta.
>
> QA: use Meta Events Manager → Test Events com o Test Event Code inserido na tag (campo adicional no HTML).

**Correspondência eventos Meta Standards:**
| Evento Canônico | Evento Meta |
|---|---|
| `Lead` | `Lead` (track padrão) |
| `MQL` | `MQL` (trackCustom) |
| `NOICP` | `NOICP` (trackCustom) |
| `DealWon` | `Purchase` (track padrão, com `value` e `currency`) |

---

### 2.3 Google Ads — Conversion Tracking Tag

**Tipo GTM:** Google Ads Conversion Tracking

```
Tag: TAG | GAds | Conversion | [EventoCanônico]
Conversion ID: {{[EDIT] Perma | GAds Conversion ID}}
Conversion Label: {{[EDIT] Perma | GAds Label | [EventoCanônico]}}
Enhanced Conversions:
  Email: {{UD | Email Hash}}
  Phone: {{UD | Phone Hash}}
Trigger: TRG | Custom Event | [EventoCanônico]
```

> Enhanced Conversions requer que os dados estejam hasheados (SHA256) e normalizados antes do envio.
> QA: verifique em Google Ads → Conversões → Diagnóstico da tag.

---

### 2.4 Clarity — Inicialização + Custom Events

**Tag de inicialização (All Pages):**
```
Tag: TAG | Clarity | Init
Tipo: HTML Customizado
```
```html
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "{{[EDIT] Perma | Clarity ID}}");
</script>
```
Trigger: All Pages

**Custom event opcional (para segmentação no Clarity):**
```html
<script>
clarity('set', 'lead_status', '{{DLV | lead_status}}');
</script>
```

---

## Seção 3 — Padrão de Nomenclatura de Tags e Gatilhos

| Elemento | Padrão | Exemplo |
|---|---|---|
| Tag GA4 | `TAG \| GA4 \| [Evento]` | `TAG \| GA4 \| Lead` |
| Tag Meta | `TAG \| Meta \| [Evento]` | `TAG \| Meta \| MQL` |
| Tag Google Ads | `TAG \| GAds \| Conversion \| [Evento]` | `TAG \| GAds \| Conversion \| Lead` |
| Tag Clarity init | `TAG \| Clarity \| Init` | — |
| Gatilho evento | `TRG \| Custom Event \| [Evento]` | `TRG \| Custom Event \| Lead` |
| Gatilho All Pages | `TRG \| All Pages` | — |

---

## Seção 4 — Checklist Pré-Publicação GTM

Antes de publicar o container, verifique:
- [ ] Todas as variáveis `[EDIT]` foram preenchidas com valores reais
- [ ] Nenhuma variável DLV tem nome divergente do dataLayer (case-sensitive)
- [ ] Todos os gatilhos usam nomes canônicos de evento
- [ ] Tags de Meta têm `eventID` mapeado para `DLV | event_id`
- [ ] Tags de Google Ads com Enhanced Conversions têm hash configurado
- [ ] Tag de inicialização Clarity está em All Pages
- [ ] QA foi executado e aprovado nas plataformas (não só no Preview)
- [ ] Não há lógica de qualificação em nenhum gatilho ou tag
