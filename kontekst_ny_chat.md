# Kontekst til ny chat: speciale om dollaren, TPU og valutaafdækning

*Overleveringsnotat, 3. november 2026. Indsæt hele dokumentet i en ny chat. Alt herunder er den gældende status. Hvor der står "afventer", er tallet eller beslutningen ikke endelig.*

---

## 0. Arbejdsregler (skal overholdes)

- **Kør aldrig koden.** Olivia kører den selv. Giv koden og forklar, hvad den gør. Claude må gerne læse filer og outputs (fx uploadede notebooks) for at tjekke tal.
- **Påstå aldrig et tal uden at have set det** i et output fra Olivia. Hvis du er i tvivl, så spørg i stedet for at gætte.
- **Skift ikke anbefaling uden ny evidens.** Hvis du ændrer en tidligere anbefaling, så sig præcis hvorfor, og hvad der var forkert.
- **Spørg hellere end at antage**, fx om hvad en kolonne eller fil er.
- Prosa skrives direkte i chatten, ikke i LaTeX, medmindre Olivia beder om det.
- **Ret ikke i hendes filer.** Ved små rettelser: giv kun det ene punkt, ikke hele filen.
- Markér ændringer med ~~gennemstregning~~ og **fed**, og giv også en ren version, der kan kopieres.
- Skriv "et al.", ikke "m.fl.". Skriv naturligt akademisk dansk uden tung eller AI-agtig stil.
- Giv altid kilder, hun kan slå op. Brug projektets papers aktivt.
- Ved figurer: brug samme formatering som de eksisterende figurer.
- Svar kort og præcist. "ALT skal være rigtigt."

---

## 1. Specialet

Olivia og Sara skriver speciale i økonomi på KU. De sigter efter 12, og afleveringen er i december 2026. Vejleder er Søren Hove Ravn.

**Problemformulering** (sendt til Søren 27. oktober 2026):
> Hvordan påvirker handelspolitisk usikkerhed den amerikanske dollars valutakurs over for euroen og dens afdækningsegenskaber, i hvilken grad har disse sammenhænge ændret sig efter Liberation Day i april 2025, og hvad kan forklare ændringen? Hvilken betydning har udviklingen for den optimale valutaafdækning af amerikanske aktiver og danske pensionskassers faktiske USD-afdækning?

**Delspørgsmål:**
1. **Dollarens valutakursrespons.** Hvordan reagerer EUR/USD på TPU-stød? Har responsen ændret sig efter april 2025, og består ændringen, når der kontrolleres for toldniveauet?
2. **Dollarens afdækningsegenskaber.** Hvordan har dollarens samvariation med amerikanske aktier og statsobligationer ændret sig efter april 2025, og dermed den optimale hedge ratio (Campbell et al. 2010)? Afhænger den af usikkerheden?
3. **Pensionskassernes faktiske afdækning.** Hvordan har danske pensionskassers USD-hedge ratio reageret på TPU før og efter april 2025? Kan kapitalkrav (Solvens II) og afdækningsomkostninger forklare forskellen mellem den faktiske og den optimale afdækning?

**Sørens svar (oktober 2026):**
- Problemformulering og delspørgsmål er fint afgrænsede.
- En egen model ville styrke specialet, men de skal fokusere på empirien nu og beslutte senere. Han nævner Galí–Monacelli-modeller med told.
- Det er acceptabelt at måle toldniveauet som provenu i forhold til import, hvis svaghederne diskuteres. Som alternativ nævner han databaser over faktiske toldsatser.

**Beslutning om model:** Byg ikke en Galí–Monacelli-model nu. En log-lineariseret model kan ikke vise en effekt af usikkerhed (certainty equivalence til første orden). Brug i stedet Bergin & Corsetti (2023) og Ostry et al. (2025) som teoretiske forudsigelser, og test dem med mekanisme-LP'en (punkt 7B). Tag beslutningen op igen ved datafreezet, og kun hvis 7A–C er færdige.

---

## 2. Data

**Placering:** Olivias computer, mappen `Speciale/Analyse/Speciale/` med undermapperne `raw/`, `clean/`, `TABLES/` og `figures/`. Der er også `raw_old/` og `clean_old/` med gamle versioner.

**Hovedfil:** `clean/master_monthly.csv` med 141 måneder fra 2015-01-31 til 2026-09-30. Kolonnerne er:
- Bloomberg: EURUSD, EURUSD_gns, USD OIS 3m, EUR OIS 3m, EURUSD basis swap 3m, SPXT, LUATTRUU, MSCIW_H, GLTREAS_H
- Usikkerhedsindeks: GPR, GPRT, GPRA, GPRC_USA, TPU
- FRED: VIXCLS, DTWEXBGS, DGS10, DGS2, INDPRO, PAYEMS, inflation
- CRSP: vwretd og bond_ret (slutter december 2025)
- Nationalbanken (DNFPVALE): eksponering og afdækning (`USD_hedge_ratio` osv.)
- Øvrige: `toldsats` og `covid`

**Ugedata:** `clean/bloomberg_weekly.csv` med 612 uger, heraf 78 efter bruddet (break-dato 2025-04-04).

**Forventede manglende værdier:**
- Markedets basis swap før februar 2021. Tickeren EUXOQQC er €STR–SOFR og starter først der.
- CRSP efter 2025.
- Toldsats for september 2026.
- `EUR_afd_usd` og `EUR_afd_ovr`.

**Pas på:** Dansk Excel viser CSV-filen forkert. Fx vises `-0.127` som `-127`, og nogle tal står som tekst. **Data er korrekte.** Tjek altid med pandas, før du påstår en datafejl.

**Toldsats:** Census "calculated duty" divideret med "customs value" (imports for consumption), i procent. Det følger Khalil & Strobel (2024) og Caldara et al. (2020). Svagheder, der skal diskuteres:
- Satsen halter efter annonceringerne.
- Importen kan blive omdirigeret.
- Importen blev fremskyndet i 1. kvartal 2025.
- Census' egne forbehold, bl.a. "Chapter 99 – no duty reported".

**DGS2:** Ligger i master som månedsgennemsnit. **Afventer:** Til mekanisme-LP'en skal der laves `DGS2_slut` og `DGS10_slut` med `.last()` (månedens sidste observation), så timingen matcher EUR/USD og OIS. Kopiér DGS10-cellen i data_cleaning. Behold `DGS10` (gennemsnit) som kontrolvariabel.

**Datafreeze ca. 15. november:** Hent Census og DNFPVALE for september 2026. Er de forsinket ud over ca. 5. november, bruges august som fælles slutmåned. Hent alt samme dag, og opdatér ikke igen. Kør derefter alle notebooks og opdatér alle tal i teksten. Tip: markér tal i LaTeX med farve, så de er lette at finde.

**Droppet:** Bloomberg-dagsdata (ugedata dækker), DXY (bredt dollarindeks er med, og DXY er ca. 58 % euro) og VIX-LP.

---

## 3. Notebooks og specifikationer

### 3.1 LP-notebooken (`LP_CLEAN-kopi.ipynb` er den aktive)

Sektioner: 1 setup, 2 data, 3 deskriptivt, 4 funktioner, 5 H_POST og lag-valg, 6 EUR/USD, 7 robusthed, 8 bootstrap, 9 hedge ratio, 10 bootstrap af bruddet, 11 diagnostik.

**Model:**
- Lag-augmenteret long-difference LP (Jordà & Taylor 2025, ligning 12; Montiel Olea & Plagborg-Møller 2021).
- Venstreside: y(t+h) − y(t−1).
- Højreside: stødet samtidigt plus `N_LAGS` lags af stødet, Δy og kontrollerne.
- HC3-standardfejl.

**Variable:**
- Stød: `lnTPU_z`, `lnGPR_z`, `lnGPRC_USA_z` (log, standardiseret).
- y: `lnUSD = −100·ln(EURUSD)`, så + betyder, at dollaren styrkes. `hr` er hedge ratio i procentpoint. `ln_afd` er afdækket beløb.
- Kontroller: `ip_growth`, `nfp_growth`, `inflation`, `d_DGS10`.

**Brud:**
- `post_LD = 1` fra 2025-04-30 (18 måneder). φ_h er interaktionen.
- Før og efter estimeres i samme model.
- H_POST er længste horisont med mindst 10 observationer efter bruddet: h = 0–8.

**Robusthed:**
- Toldsats kun lagget (`d_toldsats` som extra_control). Den samtidige toldsats er fjernet pga. risiko for bad control.
- Covid-dummy (samtidig, marts–juni 2020).
- Alternative y: `lnUSD_bred` (DTWEXBGS) og `lnUSD_gns`.
- Niveauer, fast stikprøve, uden episoder, kun til 2024, Newey-West.

**Episoder:**
- TPU: handelskrigen 2018–19 og 2025.
- GPR (reglen z > 2): feb.–mar. 2022, jun. 2025, mar.–maj 2026.

**Bootstrap:** Wild bootstrap fra VAR(p), pointwise og sup-t (simultane bånd) samt gennemsnitlig forskel.

### 3.2 Campbell-notebooken (`Campbell_ryddet.ipynb`)

- h* = 1 + β, hvor β er fra en regression af aktivets merafkast på dollarens merafkast over for euroen. Før, efter og forskel kommer fra samme model med brud-dummy.
- Merafkast bruger renten fra periodens start (lagget).
- Paneler: kvartalsafkast med Newey-West 6 lags (baseline, overlappende, brud fra 2025-06-30), månedsafkast med HC3 og ugeafkast med HC3.
- Robusthed: brud fra juni 2025 (hele kvartaler), globale indeks afdækket til USD (MSCIW_H og GLTREAS_H), CRSP og uden handelskrigen.
- Betinget δ: h*(z) = 1 + β + δz, hele perioden og før 2025, med samtidigt og lagget z.
- Rullende h* (36 måneder, W_AKTIER = 0,75).
- A_impl (implicit risikoaversion) i tre omkostningsscenarier: ingen, random walk (−ρ) og UIP (−x). Bruges kun som illustration.

### 3.3 DTV-notebooken (`Du-Tepper-Verdelhan.ipynb`)

- ρ = f − s, hvor s er log EUR pr. USD.
- Basis x = y$ − y€ + ρ.
- Afdækningsomkostning: −ρ (random walk) eller −x (UIP).
- Break-even c = omkostning / κ, med κ = 0,25 (Solvens II valutastød).

---

## 4. Beslutningen om antal lags (endelig: 3 lags)

**Hvad lag-augmentation betyder:**
- Montiel Olea & Plagborg-Møller (2021, s. 1794 og 1802): kontroller for p lags af alle variable, hvor p ≥ den sande VAR-orden p₀. Formålet er, at stødets residual efter lags bliver en innovation.
- Det er **ikke** "ét ekstra lag oven i det valgte".
- Jordà & Taylors (2025, s. 65) formulering "adding one more lag than needed" handler om AR(1)-eksemplet, hvor variablen selv indgår samtidigt.

**Proceduren** (fastlagt før kørslen):
1. AIC, HQ og BIC på VAR'en [stød, 4 kontroller, Δy] med p = 1–6 på samme stikprøve. Alle vælger 1 i alle seks kombinationer (2 y × 3 stød).
2. Breusch–Godfrey med 1 og 12 lags på stødets ligning og Δy-ligningen, uden covid-dummies. Valgt p er det mindste p, hvor ingen af testene afviser på 5 %-niveau i nogen af de seks kombinationer.

**Resultat:**
- **1 lag:** Testen med 1 lag afviser i stødets ligning i alle seks kombinationer (p = 0,006–0,012).
- **2 lags:** Testen med 1 lag afviser i Δy-ligningen for EUR/USD med GPR (p = 0,021) og GPR-USA (p = 0,018).
- **3 lags:** Ingen afvisninger. **Valgt p = 3.**

**Fejl, der ikke må gentages:**
- Ljung-Box er ugyldig, fordi stødets ligning har laggede afhængige variable. Brug Breusch–Godfrey.
- Den multivariate Portmanteau-test er ikke en del af beslutningen. Den afviser ved alle p op til 6 (396 frihedsgrader ved T = 140) og siger derfor ikke noget om antallet af lags.
- En løkke, der stopper ved PMAX, er ikke et valg.
- Covid forklarer ikke Portmanteau-afvisningerne. Den vedvarende kilde er inflationsligningen (BG12 p ≈ 0,01–0,04 ved alle p, med og uden dummies). Mulig, men ikke testet forklaring: resterende sæsonmønstre i kerneinflation målt måned for måned. Kan nævnes i en fodnote.

**Afventer (Olivia skal gøre):**
- Sæt `N_LAGS = 3` og `N_LAGS_HR = 3`, og kør hele notebooken. Standardværdierne bindes, når funktionerne defineres, og bootstrappet bliver så VAR(3).
- Robusthedsrækkerne skal være "2 lags" og "4 lags".
- Ret "VAR(2)" til "VAR(3)" i tabelnoterne.
- Erstat den første lag-valgs-celle med IC-cellen (kode i bilag), behold diagnose-cellen, tilføj appendiks-cellen, og slet Ljung-Box-cellen i afsnit 11.
- **Alle LP-tal nedenfor er fra 2-lags-kørslen og skal opdateres efter kørslen med 3 lags.**

**Tekstudkast** (indsæt de endelige tal): Se bilag D.

---

## 5. Resultater indtil videre

### 5.1 LP (2-lags-kørsel, nye data til september 2026; skal opdateres med 3 lags)

**EUR/USD og TPU:**
- Før 2025: 2,55 ved h = 3 (p = 0,004; bootstrap p = 0,006).
- φ_h er negativ ved h = 0–8: fra −2,15 til −7,18.
- Bootstrap pointwise p < 0,05 ved h = 0, 1 og 7, og p < 0,10 ved h = 6 og 8.
- Sup-t p = 0,138. Gennemsnitlig forskel −4,68 (p = 0,255).
- Uden handelskrigen er effekten før 2025 2,05 (p = 0,16). Formuleringen skal derfor være "variationen kommer især fra handelskrigen", ikke "drevet af episoden".
- Niveauspecifikation: bruddet holder kun ved h = 0–4.
- GPR på hele stikprøven ved h = 24 er ikke længere signifikant.

**Hedge ratio:**
- φ er signifikant ved h = 6 (p = 0,095) og h = 7 (p = 0,027).
- Bootstrap ikke signifikant. Sup-t p = 0,372.
- Responsen efter bruddet er signifikant ved h = 6–7.

**Afdækket beløb:**
- φ er ca. 8,5 ved h = 6–8.
- Bootstrap p = 0,032 ved h = 7. Sup-t p = 0,080.

**Til sammenligning:**
- 1 lag med nye data: EUR/USD sup-t p = 0,196. Hedge ratio φ er signifikant ved h = 6 (10 %) og 7.
- 1 lag med gamle data (til juli 2026): Hedge ratio φ er signifikant ved h = 4–6, men bootstrap har aldrig været signifikant (sup-t p = 0,438).
- **Forklaring på ændringen:** Hedge ratio-resultatet flyttede sig med de nye data (aug.–sep. 2026, hvor hedge ratioen faldt mod ca. 64 %). Skiftet i antal lags betød næsten intet.

### 5.2 Campbell (nye data; påvirkes ikke af lag-valget)

| Forskel efter april 2025, aktier | Koefficient | p |
|---|---|---|
| Kvartalsafkast (baseline) | 1,12 | 0,000 |
| Månedsafkast | 0,10 | 0,847 |
| Ugeafkast | −0,05 | 0,900 |
| Brud fra juni 2025 | 0,06 | 0,868 |
| Globale indeks | 0,98 | 0,001 |
| Uden handelskrigen | 1,14 | 0,001 |

- h* før er 0,35, h* efter er 1,47 (kvartal).
- Obligationer: bruddet er ikke signifikant (0,135; p = 0,26).
- Den gamle kørsel havde også insignifikante månedsafkast (0,65; p = 0,14).
- **Fortolkning:**
  - Kvartalsresultatet er drevet af de overlappende kvartaler, der dækker april 2025.
  - Ny fortolkning: Dollarens hedge-egenskab er uændret i gennemsnit, men den fejlede som safe haven i april 2025 (Baur & Lucey 2010). Det skal testes direkte, se punkt 7A.
- Betinget δ: δ for TPU og aktier er drevet af 2025. δ for GPR og obligationer er negativ og robust (bør tjekkes uden 2022).
- Rullende h* er meget volatil, fordi april 2025 ligger i 36-måneders-vinduet.
- A_impl-medianer: 4,6 (ingen omkostning), 8,8 (RW), 4,5 (UIP). 52–58 af 82 måneder er gyldige. Bruges kun som illustration.

### 5.3 DTV

- **Validering:** Den konstruerede basis mod markedets €STR–SOFR, 2021-02 til 2026-09 (68 måneder). Korrelation 0,99, gennemsnitlig forskel −1,6 bp, RMSE 2,8 bp. Største afvigelse: september 2026 (−8,0 mod −2,4 bp).
- **Afdækningsomkostning (RW):** 1,74 % p.a. før 2025 og 1,87 % efter april 2025. Break-even c er 6,9 % i begge perioder. Under UIP er break-even 0,9 % før og 0,1 % efter.
- **Fortolkning:** Med c = 6 % sparer afdækning 1,5 % om året i kapital (6 % × 25 %). Under RW dækker det ikke helt omkostningen. Under UIP forklarer reglerne let den høje afdækning. Konklusionen afhænger altså af, hvilke kursforventninger pensionskasserne har.
- **Basis siden 2024:** ca. 0. Omkostningen i 2025 er 2,11 %, hvoraf 2,09 % er renteforskel.
- **April 2025:** Basis udvidede sig ikke (i marts 2020 var den ca. −79 bp). Det tyder på, at der ikke var en flugt mod dollaren, og støtter tolkningen om tabt safe haven-status.
- Std. af basis 0,050 % mod std. af dollarens merafkast 3,542 % (forhold 0,014). Ret 3,569 til 3,542 i teksten.
- Solvens II-følsomhed (4,75/6/10 %) er **droppet**, fordi break-even c besvarer spørgsmålet.

### 5.4 Påstande i mailen til Søren, der ikke holder længere

- "Pensionskasserne afdækker mere, når usikkerheden stiger": gælder kun ved h = 6–7 og er ikke signifikant med bootstrap. Det var heller ikke signifikant med bootstrap med de gamle data.
- "Den optimale hedge ratio stiger markant": gælder kun med kvartalsafkast.

---

## 6. Teori og litteratur (brug aktivt)

| Kilde | Hvad den bidrager med |
|---|---|
| Ostry et al. (2025), EUI WP (`Ostry_2025.pdf`; tjek forfatterlisten) | Dollaren svækkes efter told, når andre lande gengælder. I 2018–20 faldt både 2- og 10-års renten efter toldstød. I 2025 faldt 2-års renten, men 10-års renten steg, hvilket var "unprecedented". |
| ECB Financial Stability Review (nov. 2025, boks "What safe haven after the April US tariff announcement?") | I april 2025 faldt dollaren, mens renterne steg. Korrelationen mellem amerikanske aktier og EUR/USD blev positiv, så uafdækkede investorer blev ramt. |
| Bergin & Corsetti (2023) | Optimal pengepolitik efter told: Fed sænker renten mere end andre centralbanker, og dollaren svækkes. |
| Jiang, Krishnamurthy & Lustig (2021) | Udenlandsk efterspørgsel efter sikre aktiver og convenience yield. Faldende tillid svækker dollaren. |
| Kalemli-Özcan et al. (2026) | Usikkerhed om told svækker dollaren, selvom toldniveauet i sig selv styrker den. |
| Baur & Lucey (2010), *Financial Review* 45(2) | Definitioner af hedge og safe haven samt stresstest. |
| Campbell et al. (2010), Du, Tepper & Verdelhan (2018) | Optimal afdækning; CIP-basis. |
| Montiel Olea & Plagborg-Møller (2021), *Econometrica* 89(4) | s. 1794 (definition af lag-augmentation), s. 1802 (VAR(p)), s. 1808 (bootstrap), s. 1809 (valg af p: konservativt, ingen asymptotisk omkostning). |
| Jordà & Taylor (2025), *JEL* 63(1) | s. 65 (valg af lags), s. 76 (afsnit 6.3). |
| Ivanov & Kilian (2005), *SNDE* 9(1) | AIC er bedst til impulsresponser på månedsdata. |
| Galí (2015, kap. 8), Galí & Monacelli (2005) | Fastkursdelen bruges til at argumentere for, at EUR/USD er den relevante kurs for en dansk investor. |

---

## 7. Planlagte tilføjelser (koden er givet, men endnu ikke kørt)

- **A. Safe haven-test à la Baur & Lucey på ugedata** (Campbell, ny sektion 5b). **Skal.** Konklusionen på delspørgsmål 2 hviler på den. Forvent kun ca. 5–8 stressuger efter bruddet, så gennemsnittet og listen over ugerne er vigtigere end hældningerne.
- **B. Mekanisme-LP** (LP, ny sektion 7b): renteforskel (3m OIS USD−EUR), DGS2_slut, DGS10_slut og lnSPXT, før og efter april 2025. **Skal.** Den besvarer "hvad kan forklare ændringen" og er lovet til Søren. Kræver først DGS2_slut og DGS10_slut.
  - Den pengepolitiske kanal passer, hvis renteforskellen og 2-års renten falder.
  - Tabt safe haven passer, hvis 10-års renten stiger, mens dollaren falder.
- **C. Placebo-brud** (LP, ny sektion 10b): ca. 70 falske 18-måneders-vinduer i perioden 2018-01 til 2023-10, kun med data før april 2025. Andelen er beskrivende, ikke en formel p-værdi, fordi vinduerne overlapper.
- **E. Korrelation med CRSP** (Campbell, sektion 2). Ca. 0,99 for aktier forventes. Obligationer bliver lavere, fordi løbetiderne er forskellige.

**Andre punkter, der stadig mangler:**
- Informationskriterietabel (nu en del af lag-appendikset).
- Lav tabellen `lp_gpr_2022`, eller fjern den fra teksten.
- Sæt unikke figur-labels (der er ti `\label{fig:placeholder}`), og slet skærmbillede-figuren.
- Fjern de dobbelte `rob_tpu_foer`/`rob_tpu_brud` i appendikset.
- Tilføj figuren TPU_toldsats og toldsatsens svagheder i dataafsnittet.
- Februar 2022-markeringen i hedge ratio-figuren: fjern den, eller omtal 2022 som den mekaniske kanal (hedge ratioen steg, fordi eksponeringen faldt, mens det afdækkede beløb var fladt).
- Ret "juli 2026" til september overalt. Ret "h = 6" til 8 og "10–16" til 10–18 observationer.
- Genskab afsnittet om kanaler, og skriv afsnittet med hypoteser.
- Tjek forfatterne til Staff Paper No. 5 og årstallet for Khalil & Strobel (JIE 2024).
- NB_Hedgingratio-figuren skal i afsnit 3 i LP-notebooken med Olivias oprindelige formatering. Kun `set_xlim` og `savefig` til `figures/` er ændret.

---

## Bilag: kode

### Bilag A: safe haven-test (Campbell, sektion 5b; bruger `uge`, `stjerner`, `_stj`, `gem_tex`)

```python
def tt(m, kontrast):
    t = m.t_test(kontrast)
    return float(np.squeeze(t.effect)), float(np.squeeze(t.sd)), float(np.squeeze(t.pvalue))

def safe_haven(data, q=0.10, aktiv="aktie_w", fx="usd_eur_w", brud="post_LD"):
    d = data.dropna(subset=[aktiv, fx]).copy()
    d["stress"] = (d[aktiv] <= d[aktiv].quantile(q)).astype(int)
    S, P = d["stress"], d[brud]
    d["n_foer"]  = d[aktiv] * (1 - S) * (1 - P)
    d["s_foer"]  = d[aktiv] * S * (1 - P)
    d["n_efter"] = d[aktiv] * (1 - S) * P
    d["s_efter"] = d[aktiv] * S * P
    m = smf.ols(f"{fx} ~ n_foer + s_foer + n_efter + s_efter", data=d).fit(cov_type="HC3")
    st = d[S == 1].copy()
    st["usd_pct"] = 100 * st[fx]
    g = smf.ols(f"usd_pct ~ {brud}", data=st).fit(cov_type="HC3")
    return {
        "Hældning, normale uger (hedge)":        (tt(m, "n_foer = 0"), tt(m, "n_efter = 0"), tt(m, "n_efter - n_foer = 0")),
        "Hældning, stressuger (safe haven)":     (tt(m, "s_foer = 0"), tt(m, "s_efter = 0"), tt(m, "s_efter - s_foer = 0")),
        "Gns. dollarafkast i stressuger (pct.)": (tt(g, "Intercept = 0"), tt(g, f"Intercept + {brud} = 0"), tt(g, f"{brud} = 0")),
        "_n": (int((S * (1 - P)).sum()), int((S * P).sum())),
        "_d": d}

SH = {q: safe_haven(uge, q) for q in (0.10, 0.05)}
for q, r in SH.items():
    print(f"\nStressuge = nederste {q:.0%} | stressuger før: {r['_n'][0]}, efter: {r['_n'][1]}   (før | efter | forskel)")
    for navn, celler in r.items():
        if not navn.startswith("_"):
            print(f"  {navn:40s}" + " | ".join(f"{b: .3f}{stjerner(p):3s}({s:.3f})" for b, s, p in celler))

d10 = SH[0.10]["_d"]
display(d10.loc[(d10["stress"] == 1) & (d10["post_LD"] == 1), ["date", "aktie_w", "usd_eur_w"]]
           .assign(aktie_w=lambda x: 100 * x["aktie_w"], usd_eur_w=lambda x: 100 * x["usd_eur_w"]).round(2))

linjer = []
for q, panel in [(0.10, "A"), (0.05, "B")]:
    r = SH[q]
    linjer.append(rf"\multicolumn{{4}}{{l}}{{\textit{{Panel {panel}: stressuge = nederste {int(q*100)} pct. af aktieafkastene}}}} \\")
    for navn, celler in r.items():
        if navn.startswith("_"):
            continue
        linjer.append(f"{navn} & " + " & ".join(f"${b:.2f}{_stj(p)}$" for b, s, p in celler) + r" \\")
        linjer.append(" & " + " & ".join(f"$({s:.2f})$" for b, s, p in celler) + r" \\")
    linjer.append(f"Antal stressuger & {r['_n'][0]} & {r['_n'][1]} & \\\\[4pt]")

tex = (r"""\begin{table}[H]
\centering
\caption{Dollaren som safe haven: ugeafkast i stressuger før og efter april 2025}
\label{tab:campbell_safehaven}
\begin{threeparttable}
\begin{tabular}{l ccc}
\toprule
 & Før april 2025 & Efter april 2025 & Forskel \\
\midrule
""" + "\n".join(linjer) + r"""
\bottomrule
\end{tabular}
\begin{tablenotes}[flushleft]\footnotesize
\item \textit{Note:} Ugentlige merafkast. Hældningerne er fra én regression af dollarens merafkast over for euroen på de amerikanske aktiers merafkast (S\&P 500), opdelt på normale uger og stressuger før og efter april 2025 (Baur og Lucey 2010). En negativ hældning betyder, at dollaren styrkes, når aktierne falder. En stressuge er en uge, hvor aktiernes merafkast ligger i den nederste fraktil for hele stikprøven. HC3-standardfejl i parentes. $^{*}$, $^{**}$ og $^{***}$ angiver signifikans på 10, 5 og 1 pct.-niveau.
\end{tablenotes}
\end{threeparttable}
\end{table}""")
gem_tex(tex, "campbell_safehaven.tex")
```

### Bilag B: mekanisme-LP (LP, sektion 7b; kræver `DGS2_slut` og `DGS10_slut` i master)

```python
master["renteforskel"] = master["USD OIS 3m"] - master["EUR OIS 3m"]   # pp., månedens udgang
master["lnSPXT"]       = 100 * np.log(master["SPXT"])

UDEN_D10 = ("ip_growth", "nfp_growth", "inflation")   # for renterne dækker lagget Δy det, som lagget ΔDGS10 gjorde
MEKANISME = [
    ("lnUSD",        "EUR/USD (pct.)",                "EUR/USD (pct.)",                 BASIS_KONTROLLER),
    ("renteforskel", "3m-renteforskel USD–EUR (pp.)", "3m-renteforskel USD--EUR (pp.)", UDEN_D10),
    ("DGS2_slut",    "2-årig US-rente (pp.)",         "2-årig US-rente (pp.)",          UDEN_D10),
    ("DGS10_slut",   "10-årig US-rente (pp.)",        "10-årig US-rente (pp.)",         UDEN_D10),
    ("lnSPXT",       "S&P 500 (pct.)",                r"S\&P 500 (pct.)",               BASIS_KONTROLLER)]

mek = {y: lp(master, y, "lnTPU_z", H=H_POST, dummy="post_LD", controls=k) for y, _, _, k in MEKANISME}
for y, titel, _, _ in MEKANISME:
    print(titel)
    display(mek[y][["h", "coef", "p", "coef_post", "p_post", "coef_int", "p_int", "n_post"]].round(3))

fig, akser = plt.subplots(2, 3, figsize=(15, 8.4))
for ax, (y, titel, _, _) in zip(akser.flat, MEKANISME):
    r = mek[y]
    for b, se, farve in [(r["coef"], r["se"], BLAA), (r["coef_post"], r["se_post"], ROED)]:
        ax.fill_between(r["h"], b - z95 * se, b + z95 * se, color=farve, alpha=0.15, lw=0)
        ax.plot(r["h"], b, color=farve, lw=2.5, marker="o", ms=3.5)
    ax.axhline(0, color="black", lw=0.8)
    ax.set_title(titel, fontsize=12, pad=10)
    ax.set_xticks(HOR_POST); ax.set_xlim(0, H_POST)
    ax.set_xlabel("Måneder efter stødet (h)")
    ax.spines[["top", "right"]].set_visible(False); ax.grid(axis="y", alpha=0.25)
akser.flat[-1].set_axis_off()
akser.flat[-1].legend(handles=[
    Line2D([0], [0], color=BLAA, lw=2.5, marker="o", ms=3.5, label=r"Før april 2025 ($\beta_h$)"),
    Line2D([0], [0], color=ROED, lw=2.5, marker="o", ms=3.5, label=r"Efter april 2025 ($\beta_h+\phi_h$)"),
    Patch(facecolor="dimgray", alpha=0.15, label="95 pct. KI (punktvis)")], loc="center", frameon=False, fontsize=11)
plt.tight_layout(); gem_fig("lp_mekanisme"); plt.show()

linjer = []
for y, _, titel_tex, _ in MEKANISME:
    r = mek[y].set_index("h")
    linjer.append(rf"\multicolumn{{{len(HOR_POST) + 1}}}{{l}}{{\textit{{{titel_tex}}}}} \\")
    for navn, b, se, p in [("Før", "coef", "se", "p"), ("Efter", "coef_post", "se_post", "p_post"),
                           (r"Forskel ($\phi_h$)", "coef_int", "se_int", "p_int")]:
        linjer.append(rf"\quad {navn} & " + " & ".join(f"${r.loc[h, b]:.2f}{_stj(r.loc[h, p])}$" for h in HOR_POST) + r" \\")
        linjer.append(" & " + " & ".join(f"$({r.loc[h, se]:.2f})$" for h in HOR_POST) + r" \\")
    linjer.append(r"\addlinespace")

tex = (r"""\begin{table}[H]
\centering
\footnotesize
\caption{Respons på et TPU-stød: dollar, renter og aktier før og efter april 2025}
\label{tab:lp_mekanisme}
\begin{threeparttable}
\begin{tabular}{l""" + "c" * len(HOR_POST) + r"""}
\toprule
$h$ & """ + " & ".join(str(h) for h in HOR_POST) + r""" \\
\midrule
""" + "\n".join(linjer) + r"""
\bottomrule
\end{tabular}
\begin{tablenotes}[flushleft]\footnotesize
\item \textit{Note:} Lag-augmenteret local projection som i tabel \ref{tab:lp_tpu}. Før og efter er fra samme model med brud-dummy for april 2025. For renterne udelades ændringen i den 10-årige rente fra kontrollerne, da lagget $\Delta y$ allerede indgår. Renteforskellen er 3-måneders OIS (USD minus EUR). US-renterne er månedens sidste observation (FRED). HC3-standardfejl i parentes. $^{*}$, $^{**}$ og $^{***}$ angiver signifikans på 10, 5 og 1 pct.-niveau.
\end{tablenotes}
\end{threeparttable}
\end{table}""")
gem_tex(tex, "tab:lp_mekanisme")
```

### Bilag C: placebo-brud (LP, sektion 10b)

```python
L_POST = int(master.loc[master["lnUSD"].notna(), "post_LD"].sum())      # 18
faktisk = lp(master, "lnUSD", "lnTPU_z", H=H_POST, dummy="post_LD")
phi_faktisk = faktisk["coef_int"].mean()

pre_mdr = master.loc[master["month"] < "2025-04-30", "month"].reset_index(drop=True)
MIN_FOER = 36
placebo = []
for i in range(MIN_FOER, len(pre_mdr) - L_POST + 1):
    brud_dato, slut = pre_mdr[i], pre_mdr[i + L_POST - 1]
    d = master.assign(placebo=(master["month"] >= brud_dato).astype(int))
    r = lp(d, "lnUSD", "lnTPU_z", H=H_POST, dummy="placebo", slut=slut)
    if len(r) == H_POST + 1:
        placebo.append({"brud": brud_dato, "phi_gns": r["coef_int"].mean()})
placebo = pd.DataFrame(placebo)

print(f"Faktisk gns. phi (h=0–{H_POST}): {phi_faktisk:.2f} | {len(placebo)} falske brud, "
      f"{placebo['brud'].min():%Y-%m} til {placebo['brud'].max():%Y-%m}")
print(f"Andel falske brud med gns. phi <= faktisk: {(placebo['phi_gns'] <= phi_faktisk).mean():.3f} | "
      f"med |phi| >= |faktisk|: {(placebo['phi_gns'].abs() >= abs(phi_faktisk)).mean():.3f}")

fig, ax = plt.subplots(figsize=(11, 4.6))
ax.plot(placebo["brud"], placebo["phi_gns"], color=BLAA, lw=2.2, marker="o", ms=3)
ax.axhline(phi_faktisk, color=ROED, lw=1.8, ls="--")
ax.axhline(0, color="black", lw=0.8)
ax.set_ylabel(rf"Gns. $\phi_h$ for $h$ = 0–{H_POST} (pct.)")
ax.set_xlabel(f"Dato for det falske brud (vinduet efter bruddet er {L_POST} mdr.)")
ax.legend(handles=[Line2D([0], [0], color=BLAA, lw=2.2, marker="o", ms=3, label="Falske brud"),
                   Line2D([0], [0], color=ROED, lw=1.8, ls="--", label="Faktisk brud (april 2025)")],
          frameon=False, loc="upper right")
ax.spines[["top", "right"]].set_visible(False); ax.grid(axis="y", alpha=0.25)
plt.tight_layout(); gem_fig("lp_placebo"); plt.show()
```

### Bilag D: lag-valg (LP, afsnit 5; tre celler i denne rækkefølge)

**Celle 1: informationskriterier**

```python
from statsmodels.tsa.api import VAR
PMAX, NLAGS_TEST = 6, 12

def var_data(y, shock):
    d = master.sort_values("month").reset_index(drop=True).copy()
    d["_dy"] = d[y].diff()
    return d[[shock] + list(BASIS_KONTROLLER) + ["_dy"]].dropna().reset_index(drop=True)

ic = []
for y, ynavn in [("lnUSD", "EUR/USD"), ("hr", "Hedge ratio")]:
    for s, snavn, _, _ in STOED:
        v = VAR(var_data(y, s).values).select_order(maxlags=PMAX, trend="c").selected_orders
        ic.append({"y": ynavn, "Stød": snavn, "AIC": v["aic"], "HQ": v["hqic"], "BIC": v["bic"]})
display(pd.DataFrame(ic))
```

**Celle 2: diagnose og beslutningsregel**

```python
COVID_MDR = ["2020-03-31", "2020-04-30", "2020-05-31", "2020-06-30"]

def var_data_dum(y, shock):
    d = master.sort_values("month").reset_index(drop=True).copy()
    d["_dy"] = d[y].diff()
    dum = []
    for m in COVID_MDR:
        d[f"D_{m[:7]}"] = (d["month"] == pd.Timestamp(m)).astype(float)
        dum.append(f"D_{m[:7]}")
    kol = [shock] + list(BASIS_KONTROLLER) + ["_dy"]
    d = d[kol + dum].dropna().reset_index(drop=True)
    return d[kol], d[dum]

def bg_ligninger(Y, D, p, shock, med_dummy):
    L = pd.concat({f"{c}_L{l}": Y[c].shift(l) for c in Y.columns for l in range(1, p + 1)}, axis=1)
    X = pd.concat([L, D], axis=1) if med_dummy else L
    navne = {shock: "stød", "_dy": "Δy", "ip_growth": "IP", "nfp_growth": "NFP",
             "inflation": "infl.", "d_DGS10": "ΔDGS10"}
    ud = {}
    for c in Y.columns:
        sub = pd.concat([Y[c], X], axis=1).dropna()
        m = sm.OLS(sub[c], sm.add_constant(sub[X.columns])).fit()
        if c in (shock, "_dy"):
            ud[f"BG1 {navne[c]}"] = acorr_breusch_godfrey(m, nlags=1)[1]
        ud[f"BG12 {navne[c]}"] = acorr_breusch_godfrey(m, nlags=12)[1]
    return ud

rk = []
for y, ynavn in [("lnUSD", "EUR/USD"), ("hr", "Hedge ratio")]:
    for s, snavn, _, _ in STOED:
        Y, D = var_data_dum(y, s)
        for p in (1, 2, 3):
            for med in (False, True):
                mod = VAR(Y.values, exog=D.values) if med else VAR(Y.values)
                port = mod.fit(p, trend="c").test_whiteness(nlags=NLAGS_TEST, adjusted=True).pvalue
                rk.append({"y": ynavn, "Stød": snavn, "p": p, "covid": med, "Port.": port,
                           **bg_ligninger(Y, D, p, s, med)})
diag = pd.DataFrame(rk)
pd.set_option("display.max_columns", None)
display(diag.round(3))

REGEL_KOL = ["BG1 stød", "BG12 stød", "BG1 Δy", "BG12 Δy"]
base = diag[~diag["covid"]]
ok = base.assign(bestaaet=(base[REGEL_KOL] >= 0.05).all(axis=1)).groupby("p")["bestaaet"].all()
print("Består reglen i alle seks kombinationer?\n", ok)
print("Valgt p:", int(ok[ok].index.min()) if ok.any() else "ingen af p = 1-3")
```

**Celle 3: appendikstabel**

```python
STOED_TEX = {snavn: s_tex for _, snavn, s_tex, _ in STOED}
def fmt(v):
    return rf"\textbf{{{v:.3f}}}" if v < 0.05 else f"{v:.3f}"

tab = diag[~diag["covid"]]
linjer = []
for i, (_, r) in enumerate(tab.iterrows()):
    if i > 0 and r["p"] == 1:
        linjer.append(r"\addlinespace")
    linjer.append(f"{r['y']} & {STOED_TEX[r['Stød']]} & {r['p']} & "
                  + " & ".join(fmt(r[k]) for k in REGEL_KOL) + r" \\")

tex = (r"""\begin{table}[H]
\centering
\caption{Valg af antal lags: Breusch--Godfrey-test for autokorrelation}
\label{tab:lagvalg}
\begin{threeparttable}
\begin{tabular}{ll c cc cc}
\toprule
 & & & \multicolumn{2}{c}{Stødets ligning} & \multicolumn{2}{c}{$\Delta y$-ligningen} \\
\cmidrule(lr){4-5}\cmidrule(lr){6-7}
$y$ & Stød & Lags & 1 lag & 12 lags & 1 lag & 12 lags \\
\midrule
""" + "\n".join(linjer) + r"""
\bottomrule
\end{tabular}
\begin{tablenotes}[flushleft]\footnotesize
\item \textit{Note:} $p$-værdier fra Breusch--Godfrey-tests med 1 og 12 lags på residualerne fra stødets ligning og $\Delta y$-ligningen i en VAR med stødet, de fire kontrolvariable og $\Delta y$. Fed angiver afvisning på 5 pct.-niveau. AIC, HQ og BIC vælger ét lag i alle specifikationer. Antallet af lags er det mindste, hvor ingen af testene afviser i nogen af de seks specifikationer.
\end{tablenotes}
\end{threeparttable}
\end{table}""")
gem_tex(tex, "tab:lagvalg")
```

**Tekstudkast til lag-afsnittet** (tallene er fra kørslen den 3. november 2026):

> Antallet af lags fastlægges ved en procedure, der er bestemt på forhånd. Lag-augmenteret inferens forudsætter, at stødets residual efter kontrollerne er en innovation (Montiel Olea og Plagborg-Møller, 2021, antagelse 1). Informationskriterierne (AIC, HQ og BIC) for en VAR i stødet, kontrolvariablene og $\Delta y$ vælger ét lag i alle specifikationer. Vi tester derfor for autokorrelation med Breusch--Godfrey-tests med 1 og 12 lags i stødets ligning og i $\Delta y$-ligningen og vælger det mindste antal lags, hvor ingen af testene afviser i nogen af de seks specifikationer. Med ét lag afvises hypotesen om ingen autokorrelation af første orden i stødets ligning for alle seks specifikationer ($p$-værdier mellem 0,006 og 0,012). Med to lags afvises den i EUR/USD-ligningen for de to GPR-stød ($p = 0{,}021$ og $0{,}018$). Med tre lags afvises ingen af testene (tabel \ref{tab:lagvalg}). Vi anvender derfor tre lags i alle specifikationer, hvilket også er i tråd med anbefalingen om at vælge antallet af lags konservativt (Montiel Olea og Plagborg-Møller, 2021, s. 1809). Resultaterne med to og fire lags fremgår af tabel X.

---

## Næste skridt i den nye chat

1. Olivia sætter `N_LAGS = N_LAGS_HR = 3`, retter robusthedsrækkerne til 2 og 4 lags og noterne til VAR(3), kører LP-notebooken og sender den.
2. Gennemgå de nye LP-tal mod punkt 5.1, og opdatér teksten i resultatafsnittene.
3. Kør 7A (safe haven) og 7C (placebo). Lav DGS2_slut og DGS10_slut, og kør derefter 7B (mekanisme).
4. Datafreeze ca. 15. november, derefter fuld genkørsel og opdatering af alle tal.
