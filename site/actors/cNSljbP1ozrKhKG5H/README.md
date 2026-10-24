# Brazilian CNPJ Bulk Lookup & Partner Records Scraper

Look up Brazilian companies by CNPJ in bulk, or discover companies by CNAE activity, state, city, legal nature or partner. Every result is an official-style Receita Federal registry record: legal name, status, CNAE codes, address, Simples/MEI flags, tax history, capital, and the full partner (QSA) list. Numeric and 2026 alphanumeric CNPJs are accepted, with no login, API key, or proxy.

## What this actor does

- **Bulk lookup:** paste a list, a spreadsheet column, or CSV text. CNPJs are normalized, check-digit validated, deduplicated, and resolved in one run.
- **Discovery:** find companies by CNAE (primary or secondary), state, municipality code, legal nature, or a partner's masked CPF/CNPJ, then keep only the status you want.
- **Full partner records:** name, masked document, person/company/foreign type, qualification, entry date, age band, country and legal representative.
- **Tax and status depth:** Simples Nacional, MEI, option/exclusion dates, per-year tax regime, status reason, special status and registration dates.
- **Clean output:** empty fields are omitted at every level; `false` and `0` are kept because they are real values.

## Output per company

| Field | Description |
|---|---|
| `cnpj`, `cnpjFormatado`, `cnpjRaiz` | 14-character CNPJ, formatted mask, and 8-character root |
| `razaoSocial`, `nomeFantasia` | Legal name and trade name |
| `isMatriz`, `tipoUnidade` | Headquarters or branch |
| `situacaoCadastral`, `situacaoCadastralCodigo` | Registration status and code |
| `situacaoCadastralMotivo`, `situacaoCadastralMotivoCodigo` | Reason for the current status |
| `dataSituacaoCadastral` | Date the status took effect (`YYYY-MM-DD`) |
| `situacaoEspecial`, `dataSituacaoEspecial` | Special registration status, when published |
| `cnaeFiscal`, `cnaeDescricao` | Primary CNAE code and description |
| `cnaesSecundarios` | Secondary CNAEs, each `{codigo, descricao}` |
| `naturezaJuridica`, `naturezaJuridicaCodigo` | Legal nature description and code |
| `capitalSocial` | Share capital in BRL (zero is kept) |
| `porte`, `porteCodigo` | Company size and code |
| `optanteSimples`, `dataOpcaoSimples`, `dataExclusaoSimples` | Simples Nacional flag and dates |
| `optanteMei`, `dataOpcaoMei`, `dataExclusaoMei` | MEI flag and dates |
| `regimeTributario`, `regimeTributarioAno`, `regimeTributarioHistorico` | Latest tax regime, its year, and the full per-year history |
| `endereco` | `{tipoLogradouro, logradouro, numero, complemento, bairro, municipio, codigoMunicipioIbge, uf, cep, pais, codigoPais, nomeCidadeExterior}` |
| `municipio`, `uf`, `cep` | Top-level address fields for spreadsheets |
| `telefone1`, `telefone2`, `fax`, `email` | Contact data when the registry publishes it |
| `socios` | Partner list: `{nome, documento, tipo, qualificacao, codigoQualificacao, dataEntradaSociedade, faixaEtaria, codigoFaixaEtaria, pais, codigoPais, representanteLegal}` |
| `companyAgeDays`, `statusAgeDays` | Days since activity start and since the current status |
| `source`, `sourceUrl`, `sourceUpdatedAt` | Data source, provider URL used (company endpoint for lookups, search query for discovery), and registry snapshot month |
| `recordType`, `scrapedAt` | Always `company`; ISO UTC scrape time |

**Fallback depth.** If the primary mirror and its same-schema fallback are unavailable, the independent fallback may omit the per-year tax regime and some partner detail (masked document, partner type, entry date, legal representative). Unavailable fields are omitted, never fabricated; `source` identifies the mirror that answered.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `lookup` | `lookup` or `discovery` |
| `cnpjs` | array | – | CNPJ numbers for lookup, numeric or alphanumeric, formatted or plain |
| `cnpjText` | string | – | CSV or free text containing CNPJs; valid values are extracted and deduplicated |
| `cnaes` | array | – | 7-digit CNAE codes for discovery (primary or secondary) |
| `uf` | string | `ALL` | State for discovery |
| `municipio` | string | – | IBGE/SIAFI municipality code (digits, not a city name) |
| `naturezaJuridica` | string | – | 4-digit legal-nature code |
| `partnerCpf` | string | – | Masked partner CPF/CNPJ, e.g. `***123456**` |
| `situacao` | string | `ATIVA` | `all`, `ATIVA`, `SUSPENSA`, `INAPTA`, `BAIXADA`, `NULA` |
| `includePartners` | boolean | `true` | Include the QSA list |
| `includeSecondaryActivities` | boolean | `true` | Include secondary CNAEs |
| `maxItems` | integer | `1000` | Max CNPJs processed in lookup or records emitted in discovery (1–100,000) |

### Example: bulk lookup with partners

```json
{
  "mode": "lookup",
  "cnpjs": ["18.236.120/0001-58", "00000000E08G12", "33.000.167/0001-01"],
  "includePartners": true,
  "includeSecondaryActivities": true
}
```

### Example: paste a spreadsheet column

```json
{
  "mode": "lookup",
  "cnpjText": "cliente,cnpj\nNubank,18.236.120/0001-58\nBanco do Brasil,00.000.000/E08G-12",
  "maxItems": 500
}
```

### Example: discover active companies by CNAE and city

```json
{
  "mode": "discovery",
  "cnaes": ["4713004", "6209100"],
  "uf": "SP",
  "municipio": "3550308",
  "situacao": "ATIVA",
  "maxItems": 100
}
```

### Example: find companies by a partner's masked CPF

```json
{
  "mode": "discovery",
  "partnerCpf": "***550179**",
  "uf": "RR",
  "situacao": "all",
  "maxItems": 20
}
```

## Use cases

- **KYB and vendor onboarding:** confirm a Brazilian counterparty's status, legal nature, capital and owners before signing.
- **Bulk CRM enrichment:** turn a CNPJ column into industry, address, contact and ownership data.
- **Compliance screening:** review status reason, Simples/MEI and partner structure for risk triage.
- **B2B prospecting:** build lists by CNAE, state and municipality, then filter to active companies.
- **Ownership research:** trace a masked partner across companies or review a company's QSA.
- **Market mapping:** count companies in an activity or region before outreach.

## FAQ

**What is a CNPJ?**
*Cadastro Nacional da Pessoa Jurídica* — Brazil's 14-character company registration number issued by the Receita Federal. Since July 2026 new registrations can include letters (alphanumeric CNPJ), and this actor accepts both formats.

**Where does the data come from?**
Public mirrors of the official Receita Federal CNPJ registry. The registry is published monthly, so very recent changes may not appear immediately.

**Do I need an API key, login, cookie or proxy?**
No. The actor gets real data without credentials and without any proxy setup. Just fill in the input and run it.

**What happens to invalid or unknown CNPJs?**
Invalid CNPJs are skipped before any request; unknown CNPJs are skipped by the source. The run summary reports invalid, not-found, duplicate and unavailable counts. No made-up records are ever added.

**Why is the partner list (`socios`) empty for some companies?**
MEI and individual-entrepreneur registrations normally have no QSA. The registry can also withhold a QSA in specific legal cases. The field is simply omitted rather than filled with placeholders.

**Why is `email` often missing?**
The public CNPJ registry rarely publishes company email addresses. When the source has one, it is included; otherwise the field is omitted. Phone and address data are more consistently available.

**Does `socios` include equity percentages?**
No. The public registry exposes the partner list, qualification, entry date, age band and masked document, but not each partner's percentage. Percentages are only in the company's contract at the Junta Comercial.

**Can I search by a partner's CPF?**
Yes, using the masked format shown in the QSA (for example `***123456**`). Add a state or CNAE because the same six visible digits can match more than one person.

**Is this affiliated with the Receita Federal or the Brazilian government?**
No. This is an independent, third-party actor that reads public registry mirrors and is not operated by or affiliated with any government agency.
