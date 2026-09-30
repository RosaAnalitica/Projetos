# Power BI Redesign: HTML Measures

Documentation of the HTML measures (HTML Content visual) built for the project. There is one reference measure per visual type. The full code of each measure is in `measures/`.

## 1. Design definitions (apply to every measure)

### Colors
| Use | Value |
|---|---|
| Visual background | `#E6E6E6` (on `html`, `body` and `.card`) |
| Dark blue | `#0A4F94`: highlighted headers and columns, borders, bars, line |
| Light blue | `#0078D4`: light headers, and tinted cells (`Cor` + `14` alpha) |
| Text on dark areas | `#fff` |
| Text on light areas | `#09124F` (dark navy) or `#333` |
| Chart-specific | Combo chart: bars `#F79A4A`, white background (explicit exception). Bar chart by cargo: `#5B6CE0`, white background |

### Typography and shape
- Font: `'Segoe UI', Arial, sans-serif`, weight 600 (headers 700).
- Table body: `clamp(11.7px, 2.21vh, 15.6px)`. Headers: `clamp(13px, 2.535vh, 18.2px)`. This is the base size +30%.
- Values are centered. Name and hierarchy columns are left-aligned.
- Table headers and rows are pills (`border-radius: 99px`). Cards use `border-radius: 18px`.
- Cards: the header band is dark blue, the value area is white, and the border is `2px solid #0A4F94`.
- Height: use `100vh` or `100%` so the content fills the visual with no white gap.

### Standard filters (same as the native visuals)
- `D_Usuarios[Level1]` = "Grupo Ecorodovias"
- `D_Usuarios[exclude_from_goals]` in {FALSE, BLANK}
- `D_Usuarios[Terceiro ou Não]` = BLANK
- `D_Diarios[Diarios]` excludes: Engenheiro de Obras, Engenheiro de Segurança do Trabalho, Liderança Operacional, Liderança Terceiros, Supervisor, Técnico de Obras, Técnico de Segurança do Trabalho, Engenheiro Administrativo
- `D_Cargo[cargo]` excludes: ENGENHEIRO SR, TECNICO ENGENHARIA I, TECNICO ENGENHARIA II, ENGENHEIRO PL

Remove these from the measure if they are already in the Filters pane.

### Measures used
`_Rotinas_Respondidas`, `_Rotinas_Respondidas_Switch`, `_Meta_Real_Switch`, `_Atingimento_Switch`, `_População2`, `_População_Atingida2`, `Atingimento_População`, `Atingimento_Final`, `_Escala`.

### Coding rules
1. **Config block at the top.** Colors (`Cor`, `CorDest`) and column names (`NomeColN`) are variables.
2. **No inline `style=` for data-driven values.** They get stripped. Use classes generated per row in `<style>` (`RegrasCss`), for example `.a12{width:67%}`.
3. **Do not name variables after DAX functions.** For example, `MaxA` became `LimAting`, and now `_LimAting` and `_LimResp`.
4. **Escape text.** `SUBSTITUTE` `&` to `&amp;` and `<` to `&lt;` on every text value.
5. **Decimal separator.** `SUBSTITUTE(..., ",", ".")` on numbers that go into CSS.
6. **Row index `@I`.** Built with `RANKX(..., DENSE)` and used to link rows to CSS classes and to sort.
7. **Atingimento** is assumed to be a 0-1 ratio (bars use `MIN(MAX(x,0),1)`).
8. **Text size limit.** Very long HTML strings cause the "placeholder found text too large" error (roughly 32,000 characters). Cap rows with `TOPN`, truncate long text, and keep the CSS short. Check with `LEN([Measure])`.
9. **Behavior of the HTML visual.** No JavaScript. `<input>` and `<label>` tags may be stripped, so real collapsing of a hierarchy is not reliable. A checkbox-based version made values disappear and was discarded. Only a static icon is used.

## 2. Measure catalog

| Type | Measure | File | Rows / axes | Values |
|---|---|---|---|---|
| Bar chart (ranked, horizontal bars) | `Grafico_Barras_Cargo_HTML` | `measures/Grafico_Barras_Cargo_HTML.dax` | `D_Cargo[cargo]` | `_Rotinas_Respondidas` |
| Cards | `Cards_Usuario_HTML` | `measures/Cards_Usuario_HTML.dax` | 6 boxes (Matrícula, Admissão, Diário, Cargo, Unidade, Cargo pai) | none |
| Flat matrix | `Matriz_Rotinas_HTML` | `measures/Matriz_Rotinas_HTML.dax` | `D_Rotinas[Rotinas]` and `View_DIario_Rotina_Frequencia[Frequencia]` | Respondidas, Meta, Atingimento |
| Qualitative table | `Tabela_Perguntas_HTML` | `measures/Tabela_Perguntas_HTML.dax` | `View_Perguntas_Qualitativas[question]` and `[Resposta]` | COUNT of `Question` |
| Combo chart | `Grafico_Combo_Rotinas_HTML` | `measures/Grafico_Combo_Rotinas_HTML.dax` | X: `D_Rotinas[Rotinas]` | Columns: Respondidas. Line: Atingimento |

Two more measures were built in the same format and are not in `measures/` yet: `Tabela_Usuarios_HTML` (one row per user, with a mini Atingimento bar) and `Matriz_Hierarquia_Usuarios_HTML` (Level1 to Level7 indented tree, static icon on rows with children).

## 3. Reference measures

### 3.1 Bar chart: `Grafico_Barras_Cargo_HTML`
- **Edit only the config block.** Title, color, axis column and measure.
- **Bars.** Ranked from highest to lowest. Width is relative to the maximum. Opacity fades from 1 to 0.55 by rank.
- **Value text.** Shown as `Mi` for millions, `Mil` for thousands, or the plain number.
- **Background.** White, with a colored dot before the title.

```dax
Grafico_Barras_Cargo_HTML = 

-- ===== CONFIG: only edit this block for each chart =====
VAR Titulo = "Rotinas Por Cargo"
VAR Cor    = "#5B6CE0"                          -- bar color (hex, 6 digits)
VAR Tabela =
    ADDCOLUMNS (
        VALUES ( D_Cargo[cargo] ),      -- 1) AXIS: the column here...
        "@Nome", D_Cargo[cargo],        --    ...and here (same column)
        "@V", [_Rotinas_Respondidas]            -- 2) MEASURE
    )
-- ========================================================

-- ===== Generic part: do not edit =====
VAR Dados   = FILTER ( Tabela, NOT ISBLANK ( [@V] ) && NOT ISBLANK ( [@Nome] ) )
VAR N       = COUNTROWS ( Dados )
VAR Maximo  = MAXX ( Dados, [@V] )
VAR DadosR  = ADDCOLUMNS ( Dados, "@R", RANKX ( Dados, [@V], , DESC, DENSE ) )

-- One CSS rule per rank: width + opacity live in <style>, not inline.
-- SUBSTITUTE forces a dot as the decimal separator, whatever the locale.
VAR RegrasCss =
    CONCATENATEX (
        DadosR,
        ".f" & [@R] & "{width:"
            & SUBSTITUTE ( FORMAT ( DIVIDE ( [@V], Maximo, 0 ) * 100, "0.0" ), ",", "." )
            & "%;opacity:"
            & SUBSTITUTE ( FORMAT ( 1 - 0.45 * DIVIDE ( [@R] - 1, MAX ( N - 1, 1 ) ), "0.00" ), ",", "." )
            & "}",
        "", [@R], ASC
    )

VAR Linhas =
    CONCATENATEX (
        DadosR,
        VAR Nome = SUBSTITUTE ( SUBSTITUTE ( [@Nome], "&", "&amp;" ), "<", "&lt;" )
        VAR Txt  =
            IF ( [@V] >= 1000000, FORMAT ( [@V] / 1000000, "0.0" ) & " Mi",
            IF ( [@V] >= 1000,    FORMAT ( [@V] / 1000, "0.0" ) & " Mil",
                                  FORMAT ( [@V], "0" ) ) )
        RETURN
            "<div class='r'><span class='n'>" & Nome & "</span>"
            & "<span class='t'><span class='f f" & [@R] & "'></span></span>"
            & "<span class='v'>" & Txt & "</span></div>",
        "", [@V], DESC
    )

VAR Css =
    "html,body{height:100%;margin:0;}*{box-sizing:border-box;}"
    & "body{font-family:'Segoe UI',Arial,sans-serif;background:#fff;color:#1f2430;overflow:hidden;}"
    & ".card{height:100%;display:flex;flex-direction:column;padding:1.5vh 1.5vw;}"
    & ".h{display:flex;align-items:center;gap:.6vw;flex:0 0 auto;margin-bottom:1vh;font-weight:600;font-size:clamp(11px,2.4vh,18px);}"
    & ".dot{width:.9em;height:.9em;border-radius:3px;background:" & Cor & ";}"
    & ".l{flex:1;min-height:0;overflow-y:auto;overflow-x:hidden;display:flex;flex-direction:column;}"
    & ".r{flex:1 0 24px;max-height:48px;display:flex;align-items:center;gap:1vw;padding-right:16px;font-size:clamp(9px,1.9vh,13px);}"
    & ".n{flex:0 0 32%;max-width:32%;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;color:#333;font-weight:700;}"
    & ".t{flex:1;height:38%;max-height:16px;min-width:0;background:" & Cor & "1f;border-radius:99px;overflow:hidden;}"
    & ".f{display:block;height:100%;border-radius:99px;background:" & Cor & ";}"
    & ".v{flex:0 0 12%;text-align:right;font-weight:700;white-space:nowrap;}"

RETURN
    "<html><head><meta charset='utf-8'><style>" & Css & RegrasCss & "</style></head><body><div class='card'>"
    & "<div class='h'><span class='dot'></span>" & Titulo & "</div>"
    & "<div class='l'>" & Linhas & "</div></div></body></html>"
```

### 3.2 Cards: `Cards_Usuario_HTML`
- **Layout.** Six boxes in a grid, filling `100vh`.
- **Box.** Rounded (18px) with `2px solid #0A4F94`. The header band is `#0A4F94` with white text, taking 40% of the height. The value area is white with `#09124F` text.
- **Condition.** Data shows only when exactly one user is filtered (`HASONEVALUE`). Otherwise a centered message appears.
- **Selection.** Most recent row by admission date, ties broken by matrícula.

```dax
Cards_Usuario_HTML = 

-- ===== CONFIG: colors =====
VAR Cor       = "#0078D4"      -- main blue (6-digit hex)
VAR CorDest   = "#0A4F94"      -- darker blue for labels (6-digit hex)

-- ===== CONFIG: texts (edit only the text in quotes) =====
VAR Aviso = "Por favor, filtre algum usuário"
VAR Rot1  = "Matrícula"
VAR Rot2  = "Admissão"
VAR Rot3  = "Diário"
VAR Rot4  = "Cargo"
VAR Rot5  = "Unidade"
VAR Rot6  = "Cargo pai"

-- ===== Only show data when exactly one user is filtered =====
VAR UmUsuario = HASONEVALUE ( D_Usuarios[name] )

-- Most recent row by admission date (ties: first by matrícula)
VAR UltimaData = MAX ( View_Usuario_Descritivo[data_admissao] )
VAR Linha =
    CALCULATETABLE (
        TOPN (
            1,
            VALUES ( View_Usuario_Descritivo[n_matricula] ),
            CALCULATE ( MAX ( View_Usuario_Descritivo[data_admissao] ) ), DESC,
            View_Usuario_Descritivo[n_matricula], ASC
        )
    )

VAR V1 = SUBSTITUTE ( SUBSTITUTE ( COALESCE ( CALCULATE ( MIN ( View_Usuario_Descritivo[n_matricula] ), Linha ) & "", "-" ), "&", "&amp;" ), "<", "&lt;" )
VAR V2 = COALESCE ( FORMAT ( UltimaData, "dd/mm/yyyy" ), "-" )
VAR V3 = SUBSTITUTE ( SUBSTITUTE ( COALESCE ( CALCULATE ( MIN ( View_Usuario_Descritivo[diario] ), Linha ), "-" ), "&", "&amp;" ), "<", "&lt;" )
VAR V4 = SUBSTITUTE ( SUBSTITUTE ( COALESCE ( CALCULATE ( MIN ( View_Usuario_Descritivo[cargo] ), Linha ), "-" ), "&", "&amp;" ), "<", "&lt;" )
VAR V5 = SUBSTITUTE ( SUBSTITUTE ( COALESCE ( CALCULATE ( MIN ( View_Usuario_Descritivo[unidade] ), Linha ), "-" ), "&", "&amp;" ), "<", "&lt;" )
VAR V6 = SUBSTITUTE ( SUBSTITUTE ( COALESCE ( CALCULATE ( MIN ( D_Usuarios[parent_cargo] ), Linha ), "-" ), "&", "&amp;" ), "<", "&lt;" )

VAR Css =
    "html,body{height:100vh;margin:0;background:#E6E6E6;}*{box-sizing:border-box;}"
    & "body{font-family:'Segoe UI',Arial,sans-serif;font-weight:600;background:#E6E6E6;color:#1f2430;overflow:hidden;}"
    & ".card{height:100vh;padding:1vh 1vw;background:#E6E6E6;}"
    & ".g{height:100%;display:grid;grid-template-columns:minmax(0,1fr);grid-template-rows:repeat(6,minmax(0,1fr));gap:1vh;}"
    & ".b{display:flex;flex-direction:column;min-width:0;min-height:0;overflow:hidden;border-radius:18px;background:#fff;border:2px solid #0A4F94;}"
    & ".l{flex:0 0 40%;display:flex;align-items:center;justify-content:center;font-weight:700;color:#fff;background:#0A4F94;white-space:nowrap;font-size:clamp(12px,2.88vh,21.6px);}"
    & ".v{flex:1;display:flex;align-items:center;justify-content:center;color:#09124F;background:#fff;padding:0 1vw;font-size:clamp(10.8px,3.12vh,24px);overflow:hidden;text-overflow:ellipsis;white-space:nowrap;}"
    & ".m{height:100vh;display:flex;align-items:center;justify-content:center;text-align:center;font-weight:700;color:#0A4F94;font-size:clamp(11px,2.8vh,20px);}"

VAR Caixa = "<div class='b'><span class='l'>"
VAR Meio  = "</span><span class='v'>"
VAR Fim   = "</span></div>"

VAR Grade =
    "<div class='g'>"
    & Caixa & Rot1 & Meio & V1 & Fim
    & Caixa & Rot2 & Meio & V2 & Fim
    & Caixa & Rot3 & Meio & V3 & Fim
    & Caixa & Rot4 & Meio & V4 & Fim
    & Caixa & Rot5 & Meio & V5 & Fim
    & Caixa & Rot6 & Meio & V6 & Fim
    & "</div>"

RETURN
    "<html><head><meta charset='utf-8'><style>" & Css & "</style></head><body><div class='card'>"
    & IF ( UmUsuario, Grade, "<div class='m'>" & Aviso & "</div>" )
    & "</div></body></html>"
```

### 3.3 Flat matrix: `Matriz_Rotinas_HTML`
- **Rows.** `CROSSJOIN` of `D_Rotinas[Rotinas]` and `View_DIario_Rotina_Frequencia[Frequencia]`. Only combinations with a value are shown.
- **Row key.** `@Chave` is `Rotina|Frequencia`, and `@I` is ranked on it.
- **Colors.** Rotina and Frequência are light blue. The three value columns are dark blue.
- **Atingimento.** Has a mini progress bar.
- **Note.** It is a flat list, not a collapsible matrix, with no subtotals.

```dax
Matriz_Rotinas_HTML = 

-- ===== CONFIG: colors =====
VAR Cor     = "#0078D4"      -- main blue (6-digit hex)
VAR CorDest = "#0A4F94"      -- darker blue (6-digit hex)

-- ===== CONFIG: column names (edit only the text in quotes) =====
VAR NomeCol1 = "Rotina"
VAR NomeCol2 = "Frequência"
VAR NomeCol3 = "Respondidas"
VAR NomeCol4 = "Meta"
VAR NomeCol5 = "Atingimento"

-- ===== Rows x values, with the same filters as the native visuals =====
VAR Tabela =
    CALCULATETABLE (
        ADDCOLUMNS (
            CROSSJOIN (
                FILTER ( VALUES ( D_Rotinas[Rotinas] ), NOT ISBLANK ( D_Rotinas[Rotinas] ) ),
                VALUES ( View_DIario_Rotina_Frequencia[Frequencia] )
            ),
            "@Resp",  [_Rotinas_Respondidas_Switch],
            "@Meta",  [_Meta_Real_Switch],
            "@Ating", [_Atingimento_Switch]
        ),
        TREATAS ( { "Grupo Ecorodovias" }, D_Usuarios[Level1] ),
        TREATAS ( { FALSE, BLANK () }, D_Usuarios[exclude_from_goals] ),
        TREATAS ( { BLANK () }, D_Usuarios[Terceiro ou Não] ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Diarios[Diarios] ) ),
            NOT ( D_Diarios[Diarios] IN {
                "Engenheiro de Obras", "Engenheiro de Segurança do Trabalho",
                "Liderança Operacional", "Liderança Terceiros", "Supervisor",
                "Técnico de Obras", "Técnico de Segurança do Trabalho",
                "Engenheiro Administrativo" } ) ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Cargo[cargo] ) ),
            NOT ( D_Cargo[cargo] IN {
                "ENGENHEIRO SR", "TECNICO ENGENHARIA I",
                "TECNICO ENGENHARIA II", "ENGENHEIRO PL" } ) )
    )

VAR Dados  = FILTER ( Tabela, NOT ISBLANK ( [@Resp] ) || NOT ISBLANK ( [@Meta] ) )
VAR DadosK = ADDCOLUMNS ( Dados, "@Chave", D_Rotinas[Rotinas] & "|" & View_DIario_Rotina_Frequencia[Frequencia] )
VAR DadosI = ADDCOLUMNS ( DadosK, "@I", RANKX ( DadosK, [@Chave], , ASC, DENSE ) )

-- Progress-bar width per row, in <style> (inline styles get stripped)
VAR RegrasCss =
    CONCATENATEX (
        DadosI,
        ".a" & [@I] & "{width:"
            & SUBSTITUTE ( FORMAT ( MIN ( MAX ( COALESCE ( [@Ating], 0 ), 0 ), 1 ) * 100, "0.0" ), ",", "." )
            & "%}",
        "", [@I], ASC
    )

VAR Linhas =
    CONCATENATEX (
        DadosI,
        VAR Rot = SUBSTITUTE ( SUBSTITUTE ( D_Rotinas[Rotinas], "&", "&amp;" ), "<", "&lt;" )
        VAR Fre = SUBSTITUTE ( SUBSTITUTE ( View_DIario_Rotina_Frequencia[Frequencia], "&", "&amp;" ), "<", "&lt;" )
        RETURN
            "<tr><td class='n'>" & Rot & "</td>"
            & "<td>" & Fre & "</td>"
            & "<td class='d num'>" & FORMAT ( [@Resp], "#,##0" ) & "</td>"
            & "<td class='d num'>" & FORMAT ( [@Meta], "#,##0" ) & "</td>"
            & "<td class='d at'><span class='t'><span class='f a" & [@I] & "'></span></span>"
            & "<span class='pc'>" & FORMAT ( [@Ating], "0%" ) & "</span></td></tr>",
        "", [@I], ASC
    )

VAR Css =
    "html,body{height:100%;margin:0;}*{box-sizing:border-box;}"
    & "body{font-family:'Segoe UI',Arial,sans-serif;font-weight:600;background:#E6E6E6;color:#1f2430;overflow:hidden;}"
    & ".card{height:100%;display:flex;flex-direction:column;padding:1.5vh 1.5vw;background:#E6E6E6;}"
    & ".w{flex:1;min-height:0;overflow:auto;padding-right:16px;}"
    & "table{width:100%;border-collapse:separate;border-spacing:0 4px;font-size:clamp(11.7px,2.21vh,15.6px);}"
    & "th{position:sticky;top:0;z-index:1;background:" & Cor & ";color:#fff;font-weight:700;text-align:center;padding:8px 10px;white-space:nowrap;font-size:clamp(13px,2.535vh,18.2px);}"
    & "th.d{background:" & CorDest & ";}"
    & "th:first-child{border-radius:99px 0 0 99px;}th:last-child{border-radius:0 99px 99px 0;}"
    & "td{padding:7px 10px;background:" & Cor & "14;color:#333;font-weight:600;text-align:center;}"
    & "td.d{background:" & CorDest & "1f;color:#09124F;}"
    & "td:first-child{border-radius:99px 0 0 99px;}td:last-child{border-radius:0 99px 99px 0;}"
    & ".num{white-space:nowrap;}"
    & ".at{white-space:nowrap;min-width:120px;}"
    & ".t{display:inline-block;vertical-align:middle;width:70px;height:10px;background:" & CorDest & "33;border-radius:99px;overflow:hidden;}"
    & ".f{display:block;height:100%;border-radius:99px;background:" & CorDest & ";}"
    & ".pc{display:inline-block;vertical-align:middle;margin-left:8px;}"

RETURN
    "<html><head><meta charset='utf-8'><style>" & Css & RegrasCss & "</style></head><body><div class='card'>"
    & "<div class='w'><table><thead><tr>"
    & "<th>" & NomeCol1 & "</th><th>" & NomeCol2 & "</th>"
    & "<th class='d'>" & NomeCol3 & "</th><th class='d'>" & NomeCol4 & "</th><th class='d'>" & NomeCol5 & "</th>"
    & "</tr></thead><tbody>" & Linhas & "</tbody></table></div></div></body></html>"
```

### 3.4 Qualitative table: `Tabela_Perguntas_HTML`
- **Grouping.** `SUMMARIZE` of `question` and `Resposta`, with `@Qtd = COUNT(Question)`. Rows with a count of 0 are hidden.
- **Colors.** Pergunta and Quantidade are dark blue. Resposta is light blue and left-aligned.
- **Size.** If the text is too large, cap the rows with `TOPN` and truncate long answers (see rule 8).

```dax
Tabela_Perguntas_HTML = 

-- ===== CONFIG: colors =====
VAR Cor       = "#0078D4"
VAR CorDest   = "#0A4F94"

-- ===== CONFIG: column names (edit only the text in quotes) =====
VAR NomeCol1 = "Pergunta"
VAR NomeCol2 = "Resposta"
VAR NomeCol3 = "Quantidade"

-- ===== Data (same filters as the other measures; remove if already in the Filters pane) =====
VAR Tabela =
    CALCULATETABLE (
        ADDCOLUMNS (
            SUMMARIZE (
                View_Perguntas_Qualitativas,
                View_Perguntas_Qualitativas[question],
                View_Perguntas_Qualitativas[Resposta]
            ),
            "@Qtd", CALCULATE ( COUNT ( View_Perguntas_Qualitativas[Question] ) )
        ),
        TREATAS ( { "Grupo Ecorodovias" }, D_Usuarios[Level1] ),
        TREATAS ( { FALSE, BLANK () }, D_Usuarios[exclude_from_goals] ),
        TREATAS ( { BLANK () }, D_Usuarios[Terceiro ou Não] ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Diarios[Diarios] ) ),
            NOT ( D_Diarios[Diarios] IN {
                "Engenheiro de Obras", "Engenheiro de Segurança do Trabalho",
                "Liderança Operacional", "Liderança Terceiros", "Supervisor",
                "Técnico de Obras", "Técnico de Segurança do Trabalho",
                "Engenheiro Administrativo" } ) ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Cargo[cargo] ) ),
            NOT ( D_Cargo[cargo] IN {
                "ENGENHEIRO SR", "TECNICO ENGENHARIA I",
                "TECNICO ENGENHARIA II", "ENGENHEIRO PL" } ) )
    )

VAR Dados  = FILTER ( Tabela, NOT ISBLANK ( [@Qtd] ) && [@Qtd] > 0 )
VAR DadosK = ADDCOLUMNS ( Dados, "@Chave", View_Perguntas_Qualitativas[question] & "|" & View_Perguntas_Qualitativas[Resposta] )
VAR DadosI = ADDCOLUMNS ( DadosK, "@I", RANKX ( DadosK, [@Chave], , ASC, DENSE ) )

VAR Linhas =
    CONCATENATEX (
        DadosI,
        VAR Perg = SUBSTITUTE ( SUBSTITUTE ( View_Perguntas_Qualitativas[question] & "", "&", "&amp;" ), "<", "&lt;" )
        VAR Resp = SUBSTITUTE ( SUBSTITUTE ( View_Perguntas_Qualitativas[Resposta] & "", "&", "&amp;" ), "<", "&lt;" )
        RETURN
            "<tr><td class='d n'>" & Perg & "</td>"
            & "<td class='r'>" & Resp & "</td>"
            & "<td class='d num'>" & FORMAT ( [@Qtd], "#,##0" ) & "</td></tr>",
        "", [@I], ASC
    )

VAR Css =
    "html,body{height:100%;margin:0;background:#E6E6E6;}*{box-sizing:border-box;}"
    & "body{font-family:'Segoe UI',Arial,sans-serif;font-weight:600;background:#E6E6E6;color:#1f2430;overflow:hidden;}"
    & ".card{height:100%;display:flex;flex-direction:column;padding:1.5vh 1.5vw;background:#E6E6E6;}"
    & ".w{flex:1;min-height:0;overflow:auto;padding-right:16px;}"
    & "table{width:100%;border-collapse:separate;border-spacing:0 4px;font-size:clamp(9px,1.7vh,12px);}"
    & "th{position:sticky;top:0;z-index:1;background:" & Cor & ";color:#fff;font-weight:700;text-align:center;padding:8px 10px;white-space:nowrap;font-size:clamp(10px,1.95vh,14px);}"
    & "th.d{background:" & CorDest & ";}"
    & "th:first-child{border-radius:99px 0 0 99px;}th:last-child{border-radius:0 99px 99px 0;}"
    & "td{padding:7px 14px;background:" & Cor & "14;color:#333;font-weight:600;text-align:center;}"
    & "td.d{background:" & CorDest & "1f;color:#09124F;}"
    & "td.r{text-align:left;}"
    & "td:first-child{border-radius:99px 0 0 99px;}td:last-child{border-radius:0 99px 99px 0;}"
    & ".num{white-space:nowrap;width:110px;}"

RETURN
    "<html><head><meta charset='utf-8'><style>" & Css & "</style></head><body><div class='card'>"
    & "<div class='w'><table><thead><tr>"
    & "<th class='d'>" & NomeCol1 & "</th><th>" & NomeCol2 & "</th><th class='d'>" & NomeCol3 & "</th>"
    & "</tr></thead><tbody>" & Linhas & "</tbody></table></div></div></body></html>"
```

### 3.5 Combo chart: `Grafico_Combo_Rotinas_HTML`
- **Chart.** Columns are `_Rotinas_Respondidas_Switch` on the left axis, in `#F79A4A`. The line is `_Atingimento_Switch` on the right axis, in `#0A4F94`.
- **Title.** "% Atingimento Acumulado Por Rotina", with a subtitle on each axis (`SubEsq`, `SubDir`).
- **Background.** White, as an exception to the project rule.
- **Bar values.** Inside the bars, in navy pills. All text is 40% larger than the base.
- **Axes.** Ticks sit at 0, 25, 50, 75 and 100% of the plot height, aligned with the gridlines. The left maximum rounds up to a multiple of 4 (`_LimResp`). The right maximum rounds up to a multiple of 25% (`_LimAting`).
- **Layout.** `padding-top:5.5vh` keeps the 100% markers below the title. X labels are clamped to 3 lines.
- **Categories.** Top `MaxItens` (12) Rotinas by Respondidas.

```dax
Grafico_Combo_Rotinas_HTML = 

-- ===== CONFIG =====
VAR Titulo      = "% Atingimento Acumulado Por Rotina"
VAR SubEsq      = "Rotinas respondidas"
VAR SubDir      = "% Atingimento"
VAR CorBarra    = "#F79A4A"
VAR CorLinha    = "#0A4F94"
VAR MaxItens    = 12

-- ===== Data (same filters as the native visuals) =====
VAR Tabela =
    CALCULATETABLE (
        ADDCOLUMNS (
            FILTER ( VALUES ( D_Rotinas[Rotinas] ), NOT ISBLANK ( D_Rotinas[Rotinas] ) ),
            "@Nome",  D_Rotinas[Rotinas],
            "@Resp",  [_Rotinas_Respondidas_Switch],
            "@Ating", [_Atingimento_Switch]
        ),
        TREATAS ( { "Grupo Ecorodovias" }, D_Usuarios[Level1] ),
        TREATAS ( { FALSE, BLANK () }, D_Usuarios[exclude_from_goals] ),
        TREATAS ( { BLANK () }, D_Usuarios[Terceiro ou Não] ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Diarios[Diarios] ) ),
            NOT ( D_Diarios[Diarios] IN {
                "Engenheiro de Obras", "Engenheiro de Segurança do Trabalho",
                "Liderança Operacional", "Liderança Terceiros", "Supervisor",
                "Técnico de Obras", "Técnico de Segurança do Trabalho",
                "Engenheiro Administrativo" } ) ),
        FILTER ( KEEPFILTERS ( VALUES ( D_Cargo[cargo] ) ),
            NOT ( D_Cargo[cargo] IN {
                "ENGENHEIRO SR", "TECNICO ENGENHARIA I",
                "TECNICO ENGENHARIA II", "ENGENHEIRO PL" } ) )
    )

VAR Dados0 = FILTER ( Tabela, NOT ISBLANK ( [@Resp] ) )
VAR Dados1 = ADDCOLUMNS ( Dados0, "@RN", RANKX ( Dados0, [@Nome], , ASC, DENSE ) )
VAR Dados  = TOPN ( MaxItens, Dados1, [@Resp], DESC, [@RN], ASC )
VAR DadosI = ADDCOLUMNS ( Dados, "@I", RANKX ( Dados, [@Resp] * 1000000 - [@RN], , DESC, DENSE ) )

VAR N        = COUNTROWS ( DadosI )
VAR _LimResp  = MAX ( CEILING ( MAXX ( DadosI, [@Resp] ), 4 ), 4 )
VAR _LimAting = MAX ( CEILING ( MAXX ( DadosI, [@Ating] ), 0.25 ), 0.25 )

VAR RegrasCss =
    CONCATENATEX (
        DadosI,
        VAR X = ( [@I] - 0.5 ) / N * 100
        VAR Y = 100 - DIVIDE ( COALESCE ( [@Ating], 0 ), _LimAting, 0 ) * 100
        RETURN
            ".h" & [@I] & "{height:" & SUBSTITUTE ( FORMAT ( DIVIDE ( [@Resp], _LimResp, 0 ) * 100, "0.0" ), ",", "." ) & "%}"
            & ".p" & [@I] & "{left:" & SUBSTITUTE ( FORMAT ( X, "0.00" ), ",", "." ) & "%;top:" & SUBSTITUTE ( FORMAT ( Y, "0.00" ), ",", "." ) & "%}",
        "", [@I], ASC
    )

VAR Pontos =
    CONCATENATEX (
        DadosI,
        SUBSTITUTE ( FORMAT ( ( [@I] - 0.5 ) / N * 100, "0.00" ), ",", "." ) & ","
            & SUBSTITUTE ( FORMAT ( 100 - DIVIDE ( COALESCE ( [@Ating], 0 ), _LimAting, 0 ) * 100, "0.00" ), ",", "." ),
        " ", [@I], ASC
    )

VAR Colunas =
    CONCATENATEX (
        DadosI,
        "<div class='c'><div class='bar h" & [@I] & "'><span class='bv'>" & FORMAT ( [@Resp], "#,##0" ) & "</span></div></div>",
        "", [@I], ASC
    )

VAR Marcadores =
    CONCATENATEX (
        DadosI,
        "<span class='dt p" & [@I] & "'></span><span class='pl p" & [@I] & "'>" & FORMAT ( [@Ating], "0%" ) & "</span>",
        "", [@I], ASC
    )

VAR Rotulos =
    CONCATENATEX (
        DadosI,
        "<div class='x'><span>" & SUBSTITUTE ( SUBSTITUTE ( [@Nome], "&", "&amp;" ), "<", "&lt;" ) & "</span></div>",
        "", [@I], ASC
    )

-- Axis ticks (top to bottom: 100%, 75%, 50%, 25%, 0)
VAR TicksEsq =
    "<span>" & FORMAT ( _LimResp, "#,##0" ) & "</span><span>" & FORMAT ( _LimResp * 0.75, "#,##0" )
    & "</span><span>" & FORMAT ( _LimResp * 0.5, "#,##0" ) & "</span><span>" & FORMAT ( _LimResp * 0.25, "#,##0" ) & "</span><span>0</span>"
VAR TicksDir =
    "<span>" & FORMAT ( _LimAting, "0%" ) & "</span><span>" & FORMAT ( _LimAting * 0.75, "0%" )
    & "</span><span>" & FORMAT ( _LimAting * 0.5, "0%" ) & "</span><span>" & FORMAT ( _LimAting * 0.25, "0%" ) & "</span><span>0%</span>"

VAR Css =
    "html,body{height:100vh;margin:0;background:#fff;}*{box-sizing:border-box;}"
    & "body{font-family:'Segoe UI',Arial,sans-serif;font-weight:600;color:#09124F;overflow:hidden;}"
    & ".card{height:100vh;display:flex;flex-direction:column;padding:1.5vh 1.5vw;background:#fff;}"
    & ".h{flex:0 0 auto;text-align:center;font-weight:700;color:#0A4F94;font-size:clamp(18.2px,4.06vh,30.8px);margin-bottom:1vh;}"
    & ".main{flex:1;min-height:0;display:flex;gap:.6vw;padding-top:5.5vh;}"
    & ".sub{flex:0 0 auto;writing-mode:vertical-rl;text-align:center;font-weight:700;font-size:clamp(14px,2.66vh,19.6px);display:flex;align-items:center;justify-content:center;}"
    & ".sub.e{transform:rotate(180deg);color:#B8611A;}.sub.d{color:#0A4F94;}"
    & ".axw{flex:0 0 auto;display:flex;flex-direction:column;}"
    & ".ax{flex:1;position:relative;width:3.2em;font-size:clamp(12.6px,2.38vh,16.8px);}"
    & ".ax span{position:absolute;transform:translateY(-50%);white-space:nowrap;}"
    & ".ax.e span{right:0;}.ax.d span{left:0;}"
    & ".ax span:nth-child(1){top:0;}.ax span:nth-child(2){top:25%;}.ax span:nth-child(3){top:50%;}.ax span:nth-child(4){top:75%;}.ax span:nth-child(5){top:100%;}"
    & ".sp{flex:0 0 22%;}"
    & ".mid{flex:1;min-width:0;display:flex;flex-direction:column;}"
    & ".plot{position:relative;flex:1;min-height:0;border-left:2px solid #0A4F94;border-right:2px solid #0A4F94;border-bottom:2px solid #0A4F94;"
    & "background:repeating-linear-gradient(to bottom,#d4d4d4 0,#d4d4d4 1px,transparent 1px,transparent 25%);}"
    & ".cols{position:absolute;inset:0;display:flex;}"
    & ".c{flex:1;position:relative;height:100%;}"
    & ".bar{position:absolute;bottom:0;left:22%;right:22%;background:" & CorBarra & ";border-radius:6px 6px 0 0;}"
    & ".bv{position:absolute;top:4px;left:50%;transform:translateX(-50%);background:#09124F;color:#fff;padding:0 5px;border-radius:4px;white-space:nowrap;text-align:center;font-size:clamp(11.2px,2.1vh,15.4px);font-weight:700;}"
    & ".plot svg{position:absolute;inset:0;width:100%;height:100%;}"
    & ".dt{position:absolute;width:10px;height:10px;margin:-5px 0 0 -5px;border-radius:50%;background:#fff;border:3px solid " & CorLinha & ";}"
    & ".pl{position:absolute;transform:translate(-50%,-190%);background:" & CorLinha & ";color:#fff;font-size:clamp(11.2px,2.1vh,15.4px);font-weight:700;padding:0 5px;border-radius:4px;white-space:nowrap;}"
    & ".xs{flex:0 0 22%;display:flex;}"
    & ".x{flex:1;min-width:0;display:flex;align-items:flex-start;justify-content:center;text-align:center;padding:.6vh 2px 0;font-size:clamp(13.44px,2.52vh,18.48px);line-height:1.1;}"
    & ".x span{display:-webkit-box;-webkit-line-clamp:3;-webkit-box-orient:vertical;overflow:hidden;}"

RETURN
    "<html><head><meta charset='utf-8'><style>" & Css & RegrasCss & "</style></head><body><div class='card'>"
    & "<div class='h'>" & Titulo & "</div>"
    & "<div class='main'>"
    & "<div class='sub e'>" & SubEsq & "</div>"
    & "<div class='axw'><div class='ax e'>" & TicksEsq & "</div><div class='sp'></div></div>"
    & "<div class='mid'><div class='plot'>"
    & "<div class='cols'>" & Colunas & "</div>"
    & "<svg viewBox='0 0 100 100' preserveAspectRatio='none'><polyline points='" & Pontos & "' fill='none' stroke='" & CorLinha & "' stroke-width='3' vector-effect='non-scaling-stroke'/></svg>"
    & Marcadores
    & "</div><div class='xs'>" & Rotulos & "</div></div>"
    & "<div class='axw'><div class='ax d'>" & TicksDir & "</div><div class='sp'></div></div>"
    & "<div class='sub d'>" & SubDir & "</div>"
    & "</div></div></body></html>"
```

## 4. Known limitations
- **No interactivity.** No JavaScript, no reliable collapsing, and no cross-filtering.
- **Text limit.** A large row count or long text can exceed the size limit. Use `TOPN` and truncate text.
- **Sorting.** Hierarchy rows sort alphabetically by path, not by Atingimento.
- **Ratio assumption.** Atingimento is assumed to be 0-1. If it is 0-100, change the formats and bar widths.
- **Relationships.** `View_Perguntas_Qualitativas` and `View_DIario_Rotina_Frequencia` must be related to the model, or values repeat across rows.

## 5. Checklist for a new measure
1. Copy the config block (`Cor`, `CorDest`, `NomeColN`).
2. Include the standard filters, or remove them if they are already in the Filters pane.
3. Build `Dados`, then `DadosI` with `@I`.
4. Generate the per-row CSS (`RegrasCss`) and the rows (`Linhas`).
5. Use the `#E6E6E6` background, the two blues and the standard font sizes.
6. Escape text, avoid function names as variables, and check `LEN` of the result.
