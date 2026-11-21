# Picknick Groenoordstraat

Broncode: <https://github.com/pljvp/picknick>

Twee kleine webpagina's voor de straat, plus een samenvoegtool voor elk. Alles is één bestand per pagina, draait in de browser en gebruikt geen server, geen externe diensten en geen trackers.

| Onderdeel | Bestand | Wat het doet |
|---|---|---|
| Enquête | `index.html` | Bewoners geven per parkeervak (A–M) hun mening over picknicktafels en fietsparkeren |
| Petitie | `handtekeningen.html` | Bewoners laten naam, adres en paraaf achter om het verzoek te steunen |
| Enquête samenvoegen | `tools/enquete-samenvoegen.html` | Ontvangen enquêtemails samenvoegen tot één overzicht |
| Handtekeningen samenvoegen | `tools/handtekeningen-samenvoegen.html` | Ontvangen petitiemails samenvoegen tot één lijst met naam, adres, opmerking en paraaf als plaatje. Printen of bewaren als pdf geeft een kant-en-klare lijst voor de commissie (zaaknummer, hoorzitting, voettekst). Met een vinkje kun je de paraafs onherstelbaar vervagen, voor een privacyvriendelijke versie |

```
index.html                              enquête
handtekeningen.html                     petitie
img/                                    beelden voor de petitie
tools/                                  samenvoegtools (enquête en petitie)
docs/                                   pdf's (flyer, resultaten, lijsten): alleen op de server, niet in git
../keep_local/                          enquêtedata met namen: buiten de repo of genegeerd, blijft lokaal
```

## Hoe het werkt

Beide pagina's sturen niets naar een server. Ze openen het e-mailprogramma van de bezoeker met een kant-en-klaar bericht aan het adres in `MAIL_TO` (bovenin het script van elk bestand). Werkt dat niet, dan kan de bezoeker het antwoord kopiëren en zelf versturen.

### Enquête (`index.html`)

1. De bezoeker tikt op een parkeervak op de plattegrond en kiest per vak: graag, prima, liever niet, absoluut niet of geen mening, met eventueel een toelichting.
2. Naam en huisnummer zijn optioneel, dus anoniem reageren kan. Iedereen in huis kan apart reageren. Er is ook ruimte voor algemene ideeën.
3. De pagina is in NL en EN en heeft een printversie (kaart plus vragen) voor op papier.
4. De organisatoren plakken de ontvangen mails in `tools/enquete-samenvoegen.html`, dat ze samenvoegt tot één overzicht.

### Petitie (`handtekeningen.html`)

1. De bezoeker vult naam, straat (vooraf ingevuld), huisnummer en eventueel een opmerking in, en tekent een paraaf.
2. De pagina valideert de verplichte velden en opent het e-mailprogramma met het bericht, in de taal van de bezoeker (NL of EN). Daarna volgt een bedankscherm met een deellink.
3. De organisatoren plakken de mails in `tools/handtekeningen-samenvoegen.html`. Dat levert een lijst op, gesorteerd op adres. De tool leest zowel Nederlandse als Engelse veldnamen. Print je de lijst als pdf, dan kun je met het vinkje "Handtekeningen tonen" kiezen tussen leesbare paraafs (voor de commissie) en vervaagde paraafs (om te delen).
4. De pagina waarschuwt voor misbruik (zie Privacy).

### Paraaf → code → paraaf

Een tekening in een e-mail zou groot en kwetsbaar zijn, dus de paraaf gaat als korte tekstcode mee.

- **Tekenen:** de pagina bewaart alleen de punten van elke streek (ongeveer elke 2 pixels).
- **Verkleinen:** de punten worden herschaald naar een vaste breedte van 300 en afgerond op hele getallen.
- **Verschillen:** per streek komt het startpunt, daarna alleen de stapjes van punt naar punt (dx, dy). Grote sprongen worden in stappen van maximaal 31 gesplitst.
- **Code:** elk getal wordt één teken uit een alfabet van 64 mailveilige tekens (`A–Z a–z 0–9 - _`). Formaat: `S1` + hoogte, per streek `x0 y0 aantal stappen`, en een `.` aan het eind.
- **Terug:** de samenvoegtool leest de tekens terug naar getallen, loopt de stapjes af en bouwt daaruit zelf een SVG-lijntekening (zwart op wit). Mailtekst wordt dus nooit als code in de pagina overgenomen, alleen cijfers.
- Het oude formaat (een lange `data:image/svg+xml,`-regel) wordt nog gelezen; ook daaruit worden alleen cijfers gehaald.

## Privacy

- Naam, adres, paraaf en enquêtereacties worden per e-mail verzameld en alleen gedeeld met de hoorzittingscommissie. Zo staat het ook op de petitiepagina.
- Ruwe data, ontvangen mails en pdf's met namen horen niet in deze repo. De `.gitignore` sluit `keep_local*`, `data_keep_local*`, `docs/*.pdf`, `*.csv`, `*.eml`, `*.mbox` en `*.xlsx` uit.
- Vervaagde paraafs in de pdf: de samenvoegtool verkleint elke paraaf tot een grof rooster van 10 vlakjes breed, smeert dat uit, voegt ruis toe en rondt af op weinig grijstinten. Alleen dat plaatje komt in de pdf, het origineel niet. Dit is echt informatieverlies en dus niet terug te rekenen met deblur-software, in tegenstelling tot een gewone blur. Alles gebeurt in de browser. Met `cols` bovenaan het script (`BLUR`) stel je de sterkte in: lager is vager. De vervaagde versie is getest in Chrome.
- Misbruik is strafbaar: een handtekening namaken, namens een ander tekenen of andermans gegevens invullen is valsheid in geschrifte (art. 225 Wetboek van Strafrecht). De petitiepagina waarschuwt hiervoor in het formulier en op het tabblad Privacy. De paraaf is een bewijs van betrokkenheid, geen juridisch sluitende handtekening.
- Controleer vóór elke commit met `git status --short` dat er geen persoonsgegevens tussen staan.

## Beveiliging

- Geen `innerHTML`, `eval` of externe scripts in de petitie en de handtekeningentool; alle tekst gaat via `textContent`. Controleer dit ook voor `index.html` en `tools/enquete-samenvoegen.html` als je ze aanpast.
- Content-Security-Policy in `handtekeningen.html` en `tools/handtekeningen-samenvoegen.html`: geen verbindingen naar buiten, geen externe bronnen, geen formulierverzending. Referrer wordt niet meegestuurd.
- Invoer is begrensd (opmerking maximaal 256 tekens) en gaat als platte tekst in de mail.
- De paraaf is een bewijs van betrokkenheid, geen juridisch sluitende handtekening.

## Gebruik

Zet de map op een webserver of GitHub Pages. Pas `MAIL_TO` aan in de pagina's. Zet de pdf's (enquêteresultaten en flyer) los op de server in `docs/` (de petitie linkt ernaar). De petitie heeft een hulpmiddelenvenster (tandwiel) met links naar enquête, flyer (pdf op de server), beide samenvoegtools en deze repo. Open de samenvoegtools lokaal of vanaf dezelfde server; ze verwerken alles in de browser.

## Bronvermelding

- `img/hart-leiden.svg`: de sleutels zijn afgeleid van [Flag of Leiden.svg](https://commons.wikimedia.org/wiki/File:Flag_of_Leiden.svg) door John-c, publiek domein, via Wikimedia Commons. Het hart is zelf getekend.
