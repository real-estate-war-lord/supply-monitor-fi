# Supply Monitor FI

**Paljonko uutta asuntotarjontaa on tulossa kohteiden lähelle, ja riittääkö paikallinen kysyntä siihen?**

Supply Monitor FI seuraa rakennushankkeita, vireillä olevia kaavoja ja paikallista kysyntää joko omien kohteiden (portfolio) tai minkä tahansa suomalaisen osoitteen ympärillä.

→ **Käytä:** https://real-estate-war-lord.github.io/supply-monitor-fi/

Kaikki data on julkista avointa dataa, ja se haetaan ajon aikana suoraan selaimessa. Työkalussa ei ole palvelinta, tietokantaa eikä kirjautumista. Oma kohdelista tallennetaan vain käyttäjän selaimeen.

## Näkymät

| Näkymä | Sisältö |
|---|---|
| **Portfolio** | Kohteet riskijärjestyksessä: paikallinen tarjonta vs. kysyntä, alle 6 kk ja 24 kk päähän valmistuvat, asuntokokojen osuma, kaavat ja muutokset. Lisäksi Vaatii huomiota -lista (suljettuna), valmistumiskalenteri 24 kk, kartta ja muutosloki. Vienti CSV:nä ja PDF:nä. |
| **Kohde** | Yhden kohteen kehitys: paikallinen tarjonta vs. paikallinen kysyntä, imeytyminen, hankkeet osoitteineen, kaavat, kysyntä ja laskentaoletukset. Vienti CSV:nä, JSON:na ja PDF:nä. |
| **Haku** | Sama analyysi mille tahansa osoitteelle, Google Maps -linkille, koordinaateille tai kartalta klikatulle pisteelle. |
| **✦ Claude** | Kysymykset näkymän datasta: mihin pitää varautua, tarjonta vs. kysyntä, tilannekuva komitealle. Toimii omalla API-avaimella tai "Kopioi Claudelle" -napilla. |

## Portfolion tuonti

Valitse **Portfolio: tuo / vaihda** ja anna JSON-tiedosto:

```json
{"nimi": "Oma salkku", "kohteet": [
  {"id": "k1", "nimi": "As Oy Esimerkki", "osoite": "Katu 1", "kunta": "Espoo", "kuntanumero": "049",
   "lat": 60.2, "lon": 24.7, "postinumero": "02770", "asuntoja_ryhti": 90, "keskikoko_m2": 48.2, "url": "https://…"}
]}
```

Pakollisia ovat `nimi` ja joko `lat`/`lon` tai `osoite`, jolloin koordinaatit haetaan Ryhdistä. Lista tallentuu vain selaimen localStorageen.

## Automaatio

Sivulla on rajapinta ajastetulle ajolle (esim. Claude-ajastus Chromessa):

```js
await window.SupplyMonitor.runPortfolio({ days: 7 })   // → { ok, length, chunks }
window.SupplyMonitor.chunk(0)                         // raportti ~800 merkin paloina
```

Raportissa ovat alle 6 kk päähän valmistuvat hankkeet kiinteistöittäin, muutokset, riskin top 10 ja aktiivisten hankkeiden osoitteet (esim. vuokrailmoitusdataan yhdistämistä varten).

## Datalähteet

| Tieto | Lähde |
|---|---|
| Rakennushankkeet, nykyinen asuntokanta, hankkeiden osoitteet, osoitehaku | Ryhti, Syke / ympäristöministeriö: [avoimet_lupa_rakennukset](https://paikkatiedot.ymparisto.fi/geoserver/ryhti_building/ogc/features/v1/collections/avoimet_lupa_rakennukset), avoimet_rakennukset, [open_permit_address](https://paikkatiedot.ymparisto.fi/geoserver/ryhti_permit/ogc/features/v1/collections/open_permit_address), open_address |
| Vireillä olevat kaavat | Helsinki (kaavahakemisto), Vantaa (kaavat vireillä), Espoo (kaavoitushankkeet); muut kunnat Ryhdin kaavarajapinnasta, kun tiedot on siirretty sinne |
| Paikallinen kysyntä | Tilastokeskus [Paavo](https://tilastokeskus.fi/tup/rajapintapalvelut/paavo_en.html): postinumeroalueen taloudet, asukkaat, asunnot ja vuokralla asuvat (2019→2024) |
| Kunnan kysyntä | Tilastokeskus: [väestöennuste 2024 (14wx)](https://pxdata.stat.fi/PxWeb/pxweb/fi/StatFin/StatFin__vaenn/statfin_vaenn_pxt_14wx.px/) ja [asuntokunnat (15fd)](https://pxdata.stat.fi/PxWeb/pxweb/fi/StatFin/StatFin__asku/statfin_asku_pxt_15fd.px/) |
| Karttapohja | © OpenStreetMapin tekijät |

Lähdemerkinnät:
- Lähde: Suomen ympäristökeskus (Syke), Ryhti ([CC BY 4.0](https://www.syke.fi/en/environmental-data/use-license-and-responsibilities)).
- Lähde: Tilastokeskus ([CC BY 4.0](https://stat.fi/fi/tietoa-meista/tutustu-tilastokeskukseen/lainsaadanto/kayttoehdot)).
- Kaavatiedot: Helsingin, Vantaan ja Espoon kaupungit.

## Menetelmä

- **Tarjonta** = uudisrakennus, käyttötarkoitus kerrostalo tai pientalo, huoneistoja > 0.
- **Valmistumisarvio** perustuu kunnan omaan Ryhti-historiaan (mediaaniviive, mediaanirakennusaika ja toteutumisaste). Yli 48 kk vanhat aloittamattomat luvat ovat epävarmoja, eikä niitä lasketa mukaan.
- **Valmistuminen kirjaamatta:** Ryhtiin ei aina kirjata valmistumista. Aloitettua hanketta ei lasketa tulevaan tarjontaan, jos kunnan mediaanirakennusajalla laskettu arvio on ylittynyt yli 12 kk tai jos saman kiinteistön rinnakkaislupa (päätös ±120 pv) on jo valmistunut. Hanke näkyy listassa merkinnällä "kirjaamatta?".
- **Paikallinen tarjonta vs. kysyntä** = (seuraavan 36 kk valmistumiset / 3 / nykyinen kanta säteellä) jaettuna postinumeroalueen talouksien vuosikasvulla 2019→2024. Kysynnän alaraja on 0,5 %/v, ja ↓-merkintä tarkoittaa, ettei paikallinen kysyntä kasva.
- **Imeytyminen** = talouksien lisäys / asuntojen lisäys postinumeroalueella 2019→2024. Jos imeytyminen on alle 60 %, alue ei ole niellyt edellistä tarjontaa, ja paineen pisteitä korotetaan.
- **Riskipisteet 0–100** = paine 40 + 24 kk tarjonta suhteessa kohteen asuntoihin 25 + asuntokokojen osuma (±12 m²) 20 + alle 6 kk päähän valmistuvat 15.
- **Muutokset** ovat päivämäärien perusteella näkyviä lupia, aloituksia ja valmistumisia sekä selaimessa havaittuja tilamuutoksia edelliseen tarkistukseen nähden.

## Rajoitteet

- Ryhdin kattavuus riippuu kuntien tiedonsiirrosta. Lupa- ja aloitustiedot päivittyvät hyvin, mutta valmistumisia jää kirjaamatta.
- Data ei kerro, onko kohde vuokra- vai omistusasuntokohde.
- Paavo kuvaa mennyttä kehitystä postinumerotasolla. Uusilla alueilla talouksien kasvu johtuu itse uudesta tarjonnasta, joten lue kasvua yhdessä imeytymisen kanssa.

## Tekninen

- Yksi tiedosto (`index.html`). Kirjastot: Leaflet 1.9.4 ja Chart.js 4.4.1 (cdnjs).
- Välimuistit: kunnan hankkeet ja väestöennuste 7 pv, kaavat 1 pv, Paavo 30 pv ja kohteen lähihankkeet 12 h. "↻ Päivitä data" ohittaa välimuistin.
- Paikallisesti: `python3 -m http.server`. Safari estää tiedostona avatun sivun datahaut.

## Lisenssi

Koodi: MIT (ks. `LICENSE`). Data noudattaa lähteiden omia ehtoja.
