# Brazilian CNPJ Bulk Lookup & Partner Records Scraper

Submit a list of Brazilian CNPJ numbers and get back each company's official Receita Federal record — legal name, registration status, industry codes, address, and partner list — in one structured record per CNPJ, with no limit on how many you look up in a single run.

## What this actor does

- Looks up any number of CNPJ numbers in one run, in any common formatting (with or without dots, slashes, and hyphens).
- Returns the company's registration status, size classification, legal nature, and primary/secondary industry (CNAE) codes.
- Builds a full address object plus top-level city, state, and postal code fields for each company.
- Optionally includes the partner/shareholder list (QSA) and the secondary CNAE activity codes, so you can request a smaller record when you don't need them.
- CNPJs that are invalid or not found in the registry are skipped rather than producing a per-CNPJ error record; every other field a company doesn't have on file is simply left out. If every CNPJ in a run fails to resolve, the run instead returns a small fixed set of example company records (well-known Brazilian companies) rather than an empty dataset — these are not results for your submitted CNPJs.

## Output per company

| Field | Type | Description |
|---|---|---|
| `cnpj` | string | 14-digit CNPJ number |
| `razaoSocial` | string | Official corporate name |
| `nomeFantasia` | string | Trade/brand name |
| `situacaoCadastralCodigo` | integer | Registration status code (1–8) |
| `situacaoCadastral` | string | Status label: `Ativa`, `Suspensa`, `Inapta`, `Baixada`, `Nula` |
| `dataSituacaoCadastral` | string | Date of the last status change (`YYYY-MM-DD`) |
| `porteCodigo` | integer | Company size code |
| `porte` | string | Size label: `Micro Empresa`, `Empresa de Pequeno Porte`, `Demais` |
| `tipoUnidade` | string | `Matriz` (headquarters) or `Filial` (branch) |
| `naturezaJuridica` | string | Legal nature code and description |
| `cnaeFiscal` | integer | Primary CNAE industry code |
| `cnaeDescricao` | string | Primary CNAE description |
| `cnaesSecundarios` | array | Secondary CNAE codes, each `{codigo, descricao}` (when `includeSecondaryActivities` is `true`) |
| `endereco` | object | Full address object (street, number, neighbourhood, city, state, postal code) |
| `municipio` | string | City name |
| `uf` | string | State abbreviation (e.g. `SP`, `RJ`) |
| `cep` | string | Postal code |
| `telefone1`, `telefone2` | string | Phone numbers on file, when published |
| `email` | string | Company email address, when published |
| `capitalSocial` | number | Share capital in BRL |
| `dataInicioAtividade` | string | Founding / activity start date |
| `regimeTributario` | string | Tax regime for the latest year on file |
| `socios` | array | Partner/shareholder list, each `{nome, qualificacao, faixaEtaria, dataEntradaSociedade}` (when `includePartners` is `true`) |
| `sourceUrl` | string | Direct source URL for the record |
| `recordType` | string | Always `company` |
| `scrapedAt` | string | ISO 8601 timestamp of when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `cnpjs` | array | – | CNPJ numbers to look up, formatted or plain 14-digit (required) |
| `includePartners` | boolean | `true` | Include the partner/shareholder list (QSA) in the output |
| `includeSecondaryActivities` | boolean | `true` | Include secondary CNAE industry codes in the output |

### Example: full lookup with partners and secondary activities

```json
{
  "cnpjs": ["18.236.120/0001-58", "47960950000121", "33000167000101"],
  "includePartners": true,
  "includeSecondaryActivities": true
}
```

### Example: lean lookup without partner or secondary-activity data

```json
{
  "cnpjs": ["03007331000100", "60746948000112"],
  "includePartners": false,
  "includeSecondaryActivities": false
}
```

## Use cases

- **Vendor/customer onboarding (KYB)** — confirm a Brazilian counterparty's legal status before signing a contract
- **Bulk CNPJ validation** — check thousands of CNPJs at once and keep only the ones that resolve to a real, active company
- **Compliance screening** — pull a company's legal nature, size, and registration status for a risk review
- **Sales intelligence** — enrich a prospect list with industry codes, address, and company size
- **Corporate research** — review a company's partner/shareholder list (QSA) for ownership structure

## FAQ

**What is a CNPJ?**
*Cadastro Nacional da Pessoa Jurídica* — Brazil's 14-digit national company registration number, issued by the Receita Federal (Brazilian tax authority). Every legally registered Brazilian company has one.

**Where does the data come from?**
The public BrasilAPI CNPJ endpoint, which mirrors the official Receita Federal registry.

**Do I need an API key or login?**
No — the data source is free and public, with no authentication required.

**What CNPJ formats does `cnpjs` accept?**
Any mix of formatted (`18.236.120/0001-58`) and unformatted (`18236120000158`) values in the same list.

**What happens if a CNPJ is invalid or not registered?**
It's skipped rather than producing an error record, so your output normally only contains CNPJs that actually resolved to a company. If none of the CNPJs in a run resolve, the run returns a small fixed set of example company records instead of an empty dataset — if you see unfamiliar company names you didn't request, none of your submitted CNPJs resolved.

**Is there a limit on how many CNPJs I can submit?**
No hard limit — you can submit as many as you need in one run.

**What is the `socios` (QSA) field?**
QSA stands for *Quadro Societário de Administração* — the Receita Federal's official list of a company's registered partners and administrators.

**Why would I turn off `includePartners` or `includeSecondaryActivities`?**
To get smaller, faster-to-process records when you only need the company's core identity and status, not its full ownership and industry breakdown.

**Is this affiliated with the Brazilian government or Receita Federal?**
No. This is an independent, third-party actor that reads a public mirror of Receita Federal data and is not operated by or affiliated with any government agency.
