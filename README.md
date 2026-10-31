# Plakatiprint

Ühest failist koosnev veebiäpp (`poster-tiler.html`), mis jagab suure plakati A4-lehtedeks, nii et selle saab kodus välja printida ja kokku panna. Serverit andmete töötlemiseks ei ole: kõik toimub brauseris ja printimine käib seadme enda printeri kaudu.

## Kasutamine
1. Vali pilt (PNG, JPG, WebP) või PDF, näiteks Wordist salvestatud plakat.
2. Vali valmis suurus: A3, A2, A1, A0 või oma laius. Oma laiuse puhul tuleb kõrgus pildi kujust.
3. Prindi lehed või salvesta PDF. Printimisel vali **tegelik suurus / 100%**.
4. Lõika lehed mööda roosat katkendjoont ja pane kokku. Juhend on äpis jaotises „Kokkupaneku juhend“.

Jaotises „Lisaseaded“ (vaikimisi suletud) on paigutus, sobitamine, printimata serv, ülekate, lõikejooned ja juhendi printimine.

### iPhone
Toimi kahes sammus: „Valmista printimiseks“, seejärel „Prindi kohe“ ja jagamismenüüst Print. Jagamismenüü vajab HTTPS-i, tavalise HTTP puhul avaneb PDF uuel vahekaardil. iOS-is ei saa mõõtkava valida ja lehed võivad veidi väiksemaks jääda, seega tasub täpse suuruse puhul üks leht üle mõõta.

## Paigaldamine Dockge'iga
Kopeeri `compose.yaml` ja `poster-tiler.html` kausta `/opt/stacks/poster-tiler/` ning käivita stack. nginx kuulab ainult aadressil `127.0.0.1:8088`, nii et kohtvõrgust leht kättesaadav ei ole.

Tailneti jaoks avalda see HTTPS-iga:

```bash
sudo tailscale serve --bg --https=8443 8088
```

Tailscale'i admin-konsoolis peab DNS-i all olema HTTPS lubatud. Pordi 8443 valisime sellepärast, et meie hostis kasutab porti 443 juba midagi muud.

Uuendamiseks kopeeri uus `poster-tiler.html` stacki kausta ja taaskäivita stack. `/opt/stacks` kuulub root'ile, seega on selleks vaja sudo õigusi.

## Tehniline ülevaade
- **Teegid:** PDF-i koostab jsPDF 2.5.1 ja PDF-sisendit loeb pdf.js 3.11.174. Need ja fondid tulevad CDN-ist, nii et seade vajab internetti. pdf.js-i worker on laaditud `<script>`-sildiga ja töötab põhilõimes.
- **Paigutus:** `layout()` proovib A4 mõlemat suunda ja valib selle, mis vajab vähem lehti. Samm lehtede vahel = prinditav laius − ülekate, tulpi = ceil((W − ülekate) / samm).
- **Eelvaade:** lõuendil on lehtede ruudustik, ülekatted on roosad ja lehed tähistatud A1, A2, B1… (read tähtedega, tulbad numbritega).
- **Pildikvaliteet:** 150 dpi või rohkem on terav, 100 dpi või rohkem rahuldav.
- **PDF:** iga leht renderdatakse JPEG-ina tihedusega min(200, pildi dpi × 1,25). Roosa katkendjoon on vasakul ja üleval ülekatte keskel, välja arvatud esimeses tulbas ja reas, ning sobitusmärgid on paremal ja all. PDF-is on seatud `PrintScaling: None`.
- **Printimine lauaarvutis:** PDF laaditakse peidetud iframe'i ja kutsutakse `print()`.
- **Telefonid ja tahvlid (`handheld`):** iOS lubab jagamismenüü avada ainult otse puudutusest, seepärast on printimine kaheastmeline. Sisend-PDF on piiratud 16 MP-ga, sest iOS tühjendab suuremad lõuendid, ja väljund 150 dpi-ga.
- **Claude.ai artefakt:** kui `window.claude` on olemas, peidetakse printimisnupp ja salvestamine käib läbi `claude.use("downloads")`. Mujal salvestatakse tavalise `<a download>` lingiga.

## Võimalikud edasiarendused
- Plakati suuruse määramine lehtede arvuga, näiteks 3 × 3 A4.
- Ainult valitud lehtede printimine.
- 10 cm mõõtjoon igal lehel mõõtkava kontrolliks.
