# Fixtures de laboratórios

Esta pasta contém **casos de teste de regressão** para o parser de exames do clinBoard.
Cada subpasta corresponde a um laboratório (estratégia de parser).

## Estrutura

```
fixtures/
├── README.md                  ← este arquivo
├── hoc/
│   ├── index.json             ← lista de casos de teste deste laboratório
│   ├── 1020028928/
│   │   ├── source.pdf         ← PDF original do exame
│   │   ├── extracted.txt      ← texto extraído (snapshot opcional)
│   │   └── expected.json      ← resultado esperado do parser
│   └── ...
├── piox/
└── ...
```

## Formato do `index.json`

```json
{
  "lab": "HOC",
  "description": "Hospital de Ortopedia e Ginecologia (Goiânia)",
  "cases": [
    { "id": "1020028928", "description": "Sódio + Potássio", "tags": ["bioquimica"] },
    { "id": "1020028917", "description": "TAP, Hemograma, TTPA, Bilirrubina, Ureia, Mg, Cr, Gaso, Cloro, EAS" }
  ]
}
```

## Formato do `expected.json`

Snapshot do array retornado por `parsePDFText()`. Cada entrada tem o formato:

```json
{
  "name": "Sódio",
  "value": 142.0,
  "unit": "mmol/L",
  "ref": "135 - 148",
  "refMin": 135,
  "refMax": 148,
  "date": "07/04/2026",
  "known": true
}
```

## Como adicionar um novo caso

1. Crie a pasta `fixtures/<lab>/<id>/`
2. Coloque o PDF como `source.pdf`
3. Abra o painel admin no clinBoard → "Capturar fixture" → escolha o PDF → o app gera `expected.json` automaticamente
4. Revise o `expected.json` à mão (ele é o **gabarito** — se está errado aqui, o teste vai falhar para sempre)
5. Adicione uma entrada em `index.json`
6. Commit os 3 arquivos juntos

## Como rodar os testes

No clinBoard, abra o painel admin → "Rodar testes de laboratório".
O harness:

1. Carrega `fixtures/<lab>/index.json` para cada laboratório
2. Para cada caso, baixa `source.pdf` via fetch
3. Extrai o texto, roda o parser, compara com `expected.json`
4. Mostra um relatório com pass/fail por caso e diff de campos divergentes

## Importante — proteção de dados (LGPD)

⚠️ **PDFs de fixtures NÃO vão para o git.** O `.gitignore` da raiz do projeto exclui:

- `fixtures/**/*.pdf` (PDFs de exames com dados de pacientes reais)
- `fixtures/**/expected.json` (snapshots dos resultados — também contêm valores clínicos)
- `fixtures/**/extracted.txt` (texto extraído dos PDFs)

**O que VAI para o git:**
- `fixtures/README.md` (este arquivo)
- `fixtures/<lab>/index.json` (apenas a lista de IDs e descrições, sem dados sensíveis)

**O que fica local:**
- Os PDFs em si e os `expected.json` ficam na sua máquina, dentro de `fixtures/<lab>/<id>/`. Quem clonar o repo vê a estrutura e os índices, mas precisa popular os arquivos sensíveis localmente.

**Boas práticas adicionais:**
- **Não edite `expected.json` à toa** — ele é a fonte da verdade. Se uma mudança no parser quebra um caso, ou (a) é regressão e precisa ser corrigida no parser, ou (b) é uma melhoria intencional e o `expected.json` precisa ser regenerado conscientemente.
- **Backup dos fixtures**: como eles não vão pro git, faça um backup separado (pendrive, drive criptografado, etc.) — perder os fixtures = perder a suíte de regressão.
- **Se algum dia quiser fixtures publicáveis**: anonimize manualmente um PDF (CPF, nome, data de nascimento) antes de commitar, e ajuste o `.gitignore` por exceção (`!fixtures/<lab>/sintetico-001/source.pdf`).
