# Supply Monitor FI

**Paljonko uutta asuntotarjontaa on tulossa osoitteen lähelle, ja riittääkö kysyntä siihen?**

Supply Monitor FI hakee minkä tahansa suomalaisen osoitteen ympäriltä (250 m – 2 km) kaikki luvitetut, rakenteilla olevat ja valmistuneet rakennushankkeet. Se arvioi, milloin ne valmistuvat, ja vertaa tarjonnan kasvua kunnan väestöennusteen mukaiseen asuntotarpeeseen vuoteen 2040.

→ **Käytä:** https://real-estate-war-lord.github.io/supply-monitor-fi/

Kaikki data on julkista avointa dataa, ja se haetaan ajon aikana suoraan selaimessa. Työkalussa ei ole palvelinta, tietokantaa eikä kirjautumista.

## Käyttö

- **Haku:** kirjoita osoite (`Vuotie 47 C, Helsinki`), liitä Google Mapsin pitkä linkki, anna koordinaatit (`60.207, 25.149`) tai klikkaa karttaa.
- **Jakaminen:** osoiterivin linkki avaa saman haun, esim. `…/supply-monitor-fi/#q=Keilaranta%201,%20Espoo&r=500`. Napin ⧉ painaminen kopioi linkin.
- **Tallennus:** ☆ tallentaa osoitteen pikavalintoihin (vain omaan selaimeen).

### Näkymät

| Näkymä | Sisältö |
|---|---|
| Vaatii huomiota (aina ylimpänä) | paikallinen tarjontapaine vs. kunnan kysyntä, seuraavan 24 kk valmistumiset, suurhankkeet (≥ 100 as.), kunnan tuotanto vs. tarve, vanhentuneet luvat |
| Hankkeet | kartta ja lista (rakenteilla, luvitettu, valmistunut, rauennut), CSV-vienti |
| Tarjonta ajassa | valmistuneet ja arvioidut valmistumiset vuosittain säteellä ja koko kunnassa, laskentaoletukset |
| Tarjonta vs. kysyntä | kannan kasvu vs. asuntotarpeen kasvu, väestöennuste ja 20–34-vuotiaiden kehitys |
| ✦ Claude | tarjontapaineen tulkinta, kilpailijahankkeet ja tilannekuva komitealle |

Claude-paneeli toimii joko omalla Anthropic API -avaimella, joka tallentuu vain selaimeen, tai "Kopioi Claudelle" -napilla, joka kopioi kysymyksen ja datan claude.ai:hin liitettäväksi.

## Datalähteet

| Tieto | Lähde | Rajapinta |
|---|---|---|
| Rakennushankkeet: lupa, aloitus, valmistuminen, asunnot, kerrosala | Ryhti – SYKE / ympäristöministeriö | [avoimet_lupa_rakennukset](https://paikkatiedot.ymparisto.fi/geoserver/ryhti_building/ogc/features/v1/collections/avoimet_lupa_rakennukset) |
| Nykyinen asuntokanta | Ryhti | [avoimet_rakennukset](https://paikkatiedot.ymparisto.fi/geoserver/ryhti_building/ogc/features/v1/collections/avoimet_rakennukset) |
| Osoitehaku | Ryhti, varalla OpenStreetMap Nominatim | [open_address](https://paikkatiedot.ymparisto.fi/geoserver/ryhti_building/ogc/features/v1/collections/open_address) |
| Väestöennuste 2024–2045 kunnittain | Tilastokeskus | [14wx](https://pxdata.stat.fi/PxWeb/pxweb/fi/StatFin/StatFin__vaenn/statfin_vaenn_pxt_14wx.px/) |
| Asuntokunnat ja asuntoväestö | Tilastokeskus | [15fd](https://pxdata.stat.fi/PxWeb/pxweb/fi/StatFin/StatFin__asku/statfin_asku_pxt_15fd.px/) |
| Karttapohja | OpenStreetMap (tiedostona avattuna Esri Light Gray) | – |

Lähdemerkinnät:
- Lähde: Tilastokeskus ([CC BY 4.0](https://stat.fi/fi/tietoa-meista/tutustu-tilastokeskukseen/lainsaadanto/kayttoehdot)).
- Rakennus- ja osoitetiedot: Ryhti, Suomen ympäristökeskus ja ympäristöministeriö. Ryhdin avoimille rakennustiedoille ei ole julkaistu erillistä lisenssitekstiä. Tarkemmat käyttöehdot: ryhti@syke.fi.
- Karttapohja © OpenStreetMapin tekijät.

## Menetelmä

- **Tarjonta** = uudisrakennus (`Uusi rakennus tai rakennelma`), käyttötarkoitus kerrostalo tai pientalo, huoneistoja > 0. Laajennukset näkyvät listassa, mutta niitä ei lasketa tarjontaan, ja muutostyöt on rajattu pois.
- **Valmistumisarvio** perustuu kunnan omaan Ryhti-historiaan:
  - rakenteilla = aloitus + mediaanirakennusaika
  - luvitettu = lupapäätös + mediaaniviive + rakennusaika, painotettuna toteutumisasteella (3–6 v sitten luvitetuista asunnoista aloitettu osuus)
  - yli 48 kk vanhat aloittamattomat luvat merkitään epävarmoiksi, eikä niitä lasketa ennusteeseen
- **Asuntotarve 2025→2040** = asuntokunnat 2025 × (väkiluku 2040 / 2025 − 1). Yläraja olettaa, että asuntokunnan koon 2015–2025 trendi jatkuu.
- **Paikallinen paine** = seuraavan 36 kk valmistumiset / 3 / nykyinen kanta säteellä, verrattuna kunnan vuotuiseen tarpeeseen / asuntokunnat. Suhde > 1 tarkoittaa, että lähialueen kanta kasvaa kunnan kysyntää nopeammin.
- **Validointi:** Ryhdistä laskettu Helsingin valmistuneiden asuntojen määrä 2025 oli 4 119. Kaupungin oma luku on 4 186 ([hel.fi](https://www.hel.fi/fi/uutiset/rakentamisen-hiljentyminen-vaikeuttaa-edelleen-helsingin-asuntotuotantotavoitteiden-saavuttamista)).

## Rajoitteet

- Ryhdin kattavuus riippuu kuntien tiedonsiirrosta.
- Asemakaavavaiheen hankkeet, joilla ei vielä ole rakennuslupaa, eivät näy.
- Data ei kerro, onko kohde vuokra- vai omistusasuntokohde, eikä rakennuttajaa.
- Väestöennuste on kuntatasoinen.

## Tekninen

- Yksi tiedosto (`index.html`). Kirjastot: Leaflet 1.9.4 ja Chart.js 4.4.1 (cdnjs).
- Kunnan hankeaineisto välimuistitetaan selaimeen 7 päiväksi. Ensimmäinen haku suuressa kunnassa vie noin 10 s, ja "↻ Päivitä data" ohittaa välimuistin.
- Paikallisesti: `python3 -m http.server` ja selaimella http://localhost:8000. Tiedostona avattuna toimii Chromessa, mutta Safari estää levyltä avatun sivun datahaut.
- Teemat: Energy Manager, Neutral Mono (60-30-10) ja CapMan.

## Lisenssi

Koodi: MIT (ks. `LICENSE`). Data noudattaa lähteiden omia ehtoja.
