---
title: "Documenten API"
date: '28-6-2022'
weight: 10
layout: page-with-side-nav
---
# Documenten API

API voor opslag en ontsluiting van documenten en daarbij behorende metadata.

De API ondersteunt het opslaan en naar andere applicaties ontsluiten van informatieobjecten (in de 'volksmond': documenten). De component slaat deze gestructureerd en voorzien van de benodigde metadata op en stelt applicaties in staat deze te wijzigen, te verwijderen en aan de hand van een aantal zoekcriteria op te vragen. Opslag vindt plaats conform het informatie-objecten-gedeelte van het RGBZ.


## Gegevensmodel

Een informatieobject is een generiekere term voor het veelgebruikte begrip document dat beperkter van reikwijdte is.

Een informatieobject kan van alles zijn, ongeacht aard en vorm: een tekstverwerkingsdocument, een papieren brief, een webpagina, een landkaart, een foto, een geluidsopname, een dataset, een blog, etc.

### Enkelvoudige en samengestelde informatieobjecten

Vooralsnog ondersteunt de Documenten API alleen enkelvoudige informatieobjecten. Een e-mail met drie bijlagen of een verzoek met bijbehorende CAD-tekening en Excel spreadsheet kan worden beschouwd als een samengesteld informatieobject. Mogelijk dat dit objecttype in de toekomst nog wordt toegevoegd aan deze API.

### Relatie met zaken en besluiten

Een informatieobject kan tot meer dan één zaak behoren en een zaak kan meer dan één informatieobjecten bevatten. De relatie tussen zaak en informatieobject is vastgelegd in zaakinformatieobject (Zaken API) en objectinformatieobject (Documenten API), waarbij zaakinformatieobject leidend is.

Een besluit kan vastgelegd zijn in een informatieobject. De relatie tussen besluit en informatieobject is vastgelegd in besluitinformatieobject (Besluiten API) en objectinformatieobject (Documenten API), waarbij besluitinformatieobject leidend is.

### Opslaan van bestanden
In versie 1.0.x van de API moet het bestand base64-encoded opgeslagen worden in het attribuut `inhoud`. De omvang mag 4GB groot zijn. Hou hierbij rekening met de overhead van base64, die, in worst-case scenario's, ongeveer 33% bedraagt . Dit betekent dat bij een limiet van 4GB het bestand maximaal ongeveer 3GB groot mag zijn.

<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>Nieuw in versie 1.1.0</strong>
</span>
Afhankelijk van de omvang van het bestand wordt de inhoud van het informatieobject als volgt opgeslagen:
* omvang 0: het attribuut `inhoud` blijft leeg
* kleine omvang: in het attribuut `inhoud`.
* grote omvang: via aparte `bestandsdelen`

### Archief- en dossiervorming

Alle informatieobjecten van de zaak vormen het zaakarchief, de informatieobjecten en zaakkenmerken samen vormen het zaakdossier.

[![Gegevensmodel Documenten API 1.0.0](Documenten API.png)](Documenten API.png "Documenten gegevensmodel - klik voor groot")

[![Gegevensmodel Documenten API 1.1.0](Documenten API 1.1.0.png)](Documenten API 1.1.0.png "Documenten gegevensmodel - klik voor groot")

### Verzoekinformatieobjecten

<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>Nieuw in versie 1.1.0</strong>
</span>
Een verzoek kan onderbouwd worden met één of meer informatieobjecten. De relatie tussen verzoek en informatieobject is vastgelegd in verzoekinformatieobject (Verzoeken API) en objectinformatieobject (Documenten API), waarbij verzoekinformatieobject leidend is.

### Verzending

<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>Nieuw in versie 1.2.0</strong>
</span>
De relatie klasse Verzending legt vast aan welke Betrokkene een Informatieobject verzonden is of van welke Betrokkene een Informatieobject ontvangen is. Om altijd te kunnen achterhalen naar/van welk adres een Informatieobject verzonden of ontvangen is moet dit adres ook worden vastgelegd. Immers, wanneer alleen verwezen wordt naar het adres waarop iemand ingeschreven staat verandert dit gegeven wanneer deze persoon verhuist of de geregistreerde gegevens bijgewerkt worden. Door het adres vast te leggen in Verzending is altijd te achterhalen naar/van welk adres een Informatieobject verstuurd/ontvangen is. 

Het attribuut richting uit de relatieklasse ZaaktypeInformatieobjecttype is hiermee overbodig en deprecated geworden.


## Specificatie van de Documenten API

* [Referentie-implementatie Documenten API](https://documenten-api.vng.cloud/)
* API specificatie (OAS3) in
  [ReDoc][documenten-1.0.1-redoc],
  [Swagger][documenten-1.0.1-swagger],
  [YAML](https://documenten-api.vng.cloud/api/v1/schema/openapi.yaml) of
  [JSON](https://documenten-api.vng.cloud/api/v1/schema/openapi.json)

[documenten-1.0.1-redoc]: /gemma-zaken/content/standaard/documenten/redoc-1.0.1
[documenten-1.0.1-swagger]: /gemma-zaken/content/standaard/documenten/swagger-ui-1.0.1

## Specificatie van gedrag

Documenten APIsen (DRC) MOETEN aan twee aspecten voldoen:

* de DRC `openapi.yaml` MOET volledig geïmplementeerd zijn.

* het run-time gedrag beschreven in deze standaard MOET correct geïmplementeerd   zijn.

### OpenAPI specificatie

Alle operaties beschreven in [`openapi.yaml`](https://documenten-api.vng.cloud/api/v1/schema/openapi.yaml) MOETEN ondersteund worden en tot hetzelfde resultaat leiden als de referentie-implementatie van het DRC.

Het is NIET TOEGESTAAN om gebruik te maken van operaties die niet beschreven staan in deze OAS spec, of om uitbreidingen op operaties in welke vorm dan ook toe te voegen.

### Run-time gedrag

Bepaalde gedrageningen kunnen niet in een OAS spec uitgedrukt worden omdat ze businesslogica bevatten. Deze gedragingen zijn hieronder beschreven en MOETEN zoals beschreven geïmplementeerd worden.

#### **<a name="drc-001">Valideren `informatieobjecttype` op de `EnkelvoudigInformatieObject`-resource ([drc-001](#drc-001))</a>**

Bij het aanmaken (`enkelvoudiginformatieobject_create`) MOET de URL-referentie naar het `informatieobjecttype` gevalideerd worden op het bestaan. Indien het ophalen van het informatieobjecttype niet (uiteindelijk) resulteert in een `HTTP 200` status code, MOET het DRC antwoorden met een `HTTP 400` foutbericht.

De provider MOET tevens valideren dat het opgehaalde informatieobjecttype een informatieobjecttype is conform de geldige Catalogi API specificatie.

<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>overgenomen uit de openapi.yaml</strong>
</span>
Daarnaast MOET de provider valideren dat het opgehaalde 'informatieobjecttype' 'concept = false' is. Indien het opgehaalde 'informatieobjecttype' niet 'concept = false' is MOET de DRC antwoorden met een `HTTP 400` foutbericht.

Als er geprobeerd wordt om het `informatieobjecttype` van een bestaand `EnkelvoudigInformatieObject` bij te werken (`enkelvoudiginformatieobject_update`, `enkelvoudiginformatieobject_partial_update`), dan MOET het DRC antwoorden met een `HTTP 400` foutbericht.

#### **<a name="drc-002">Valideren `object` op de `ObjectInformatieObject`-resource ([drc-002](#drc-002))</a>**

Bij het aanmaken (`objectinformatieobject_create`) MOET de URL-referentie naar het `object` gevalideerd worden op het bestaan. Indien het ophalen van het object niet (uiteindelijk) resulteert in een `HTTP 200` status code, MOET het DRC antwoorden met een `HTTP 400` foutbericht.

(TODO: valideren dat het van het type `object_type` is -> validatie aanscherpen)

#### **<a name="drc-003">Valideren uniciteit combinatie `object` en `informatieobject` op de `ObjectInformatieObject`-resource ([drc-003](#drc-003))</a>**

Er MOET gevalideerd worden dat de combinatie `object` en `informatieobject` niet eerder voorkomt. Indien deze al bestaat, dan MOET het DRC antwoorden met een `HTTP 400` foutbericht.

#### **<a name="drc-004">Valideren bestaan relatie tussen `object` en `informatieobject` in de bron ([drc-004](#drc-004))</a>**

Er MOET gevalideerd worden dat de relatie tussen het `object` en het `informatieobject` al bestaat in de bron van het `object`. De bron van het informatieobject is bekend door de eerdere validaties op deze URL. De API-spec van het bron register voorziet in query-parameters om het bestaan te kunnen valideren.

#### **<a name="drc-005">Statuswijzigingen van informatieobjecten ([drc-005](#drc-005))</a>**

Wanneer `InformatieObject.ontvangstdatum` een waarde heeft, dan zijn de waarden `in bewerking` en `ter vaststelling` voor `InformatieObject.status` NIET TOEGESTAAN. Indien een dergelijke status gezet is _voor_ de verzenddatum opgegeven wordt, dan moet de API een HTTP 400 foutbericht geven met `status` als veld in de `invalid-params`. De client MOET dan `ontvangstdatum` leeg laten of eerst de status wijzingen.

#### **<a name="drc-006">Gebruiksrechten op informatieobjecten ([drc-006](#drc-006))</a>**

Indien er geen gebruiksrechtenvoorwaarden van toepassing zijn op een informatieobject, dan moet `InformatieObject.indicatieGebruiksrechten` op de waarde `false` gezet worden. Indien de voorwaarden (nog) niet bekend zijn, dan moet de indicatie op `null` gezet worden.

Om de indicatie op `true` te zetten, MOET je de resource `Gebruiksrechten` aanmaken in de API. Providers MOETEN bij het aanmaken van gebruiksrechten voor een informatieobject de `indicatieGebruiksrechten` van dat informatieobject op `true` zetten.

Indien de laatste gebruiksrechten op een informatieobject verwijderd worden, dan MOET de indicatie weer op `null` gezet worden.

#### **<a name="drc-007">Vertrouwelijkheidaanduiding van een informatieobject ([drc-007](#drc-007))</a>**

Indien de client een `vertrouwelijkheidaanduiding` meegeeft bij het aanmaken of bewerken van een informatieobject, dan MOET de provider deze waarde toekennen. Indien de client deze niet expliciet toekent, dan MOET deze afgeleid worden uit `InformatieOject.InformatieObjectType.vertrouwelijkheidaanduiding`.

Een `InformatieOject` response van de provider MOET altijd een geldige waarde voor `vertrouwelijkheidaanduiding` bevatten. Een client MAG een waarde voor `vertrouwelijkheidaanduiding` meesturen.

#### Archiveren

**<a name="drc-008">Vernietigen van informatieobjecten ([drc-008](#drc-008))</a>**

Een `EnkelvoudigInformatieObject` MAG ALLEEN verwijderd worden indien er geen `ObjectInformatieObject`-en meer aan hangen. Indien er nog relaties zijn, dan MOET het DRC antwoorden met een `HTTP 400` foutbericht

Bij het verwijderen van een `EnkelvoudigInformatieObject` MOETEN het `EnkelvoudigInformatieObject` en gerelateerde objecten daadwerkelijk uit de opslag verwijderd worden. Zogenaamde "soft-deletes" zijn NIET TOEGESTAAN. Onder gerelateerde objecten wordt begrepen:
- `gebruiksrechten` - de gebruiksrechten die horen bij het `EnkelvoudigInformatieObject`.
- `audittrail` - de geschiedenis van het object.

#### **<a name="drc-009">Locken en unlocken van documenten ([drc-009](#drc-009))</a>**

Bij het bijwerken van `InformatieObject` (`enkelvoudiginformatieobject_update`, `enkelvoudiginformatieobject_partial_update`) MOET eerst een `lock` verkregen worden. De consumer voert de `enkelvoudiginformatieobject_lock` operatie uit, waarbij het DRC MOET antwoorden met een niet-te-raden `lockId`. Het DRC MOET vervolgens alle schrijf-operaties blokkeren tenzij het correcte `lockId` meegegeven is.

Het DRC MOET geforceerd unlocken toelaten door 'administrators'. Dit zijn applicaties die de scope `documenten.geforceerd-unlock` hebben. Deze consumers MOETEN het `lockId` weglaten indien ze geforceerd unlocken.

#### **<a name="drc-010">Bijwerken van documenten ([drc-010](#drc-010))</a>**
<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>overgenomen uit de openapi.yaml</strong>
</span>
Bij het werken wordt gevalideerd of:
- Een correcte lock waarde aanwezig is (zie ([drc-009](#drc-009))
- De  status NIET definitief is
- Het informatieobjecttype niet gewijzigd wordt

Wanneer aan één of meer van deze voorwaarden niet wordt voldaan MOET het DRC antwoorden met een `HTTP 400` foutbericht. 


#### HTTP-Caching

<span style="padding: 0.2em 0.5em; border: solid 1px #EEEEEE; border-radius: 3px; background: #DDDFFF;">
    <strong>Nieuw in versie 1.1.0</strong>
</span>

De Documenten API moet HTTP-Caching ondersteunen op basis van de `ETag` header. In de API spec staat beschreven voor welke resources dit van toepassing is.

De `ETag` MOET worden berekend op de JSON-weergave van de resource. Verschillende, maar equivalente weergaves (bijvoorbeeld dezelfde API ontsloten wel/niet via NLX) MOETEN verschillende waarden voor de `ETag` hebben.

Indien de consumer een `HEAD` verzoek uitvooert op deze resources, dan MOET de provider antwoorden met dezelfde headers als bij een normale `GET`, dus inclusief de `ETag` header. Er MAG GEEN response body voorkomen.

Indien de consumer gebruik maakt van de `If-None-Match` header, met één of meerdere waarden voor de `ETag`, dan MOET de provider antwoorden met een `HTTP 304` bericht indien de huidige `ETag` waarde van de resource hierin voorkomt. Als de huidige `ETag` waarde hier niet in voorkomt, dan MOET de provider een normale `HTTP 200` response sturen.

## Overige documentatie

* [Referentiemodel Gemeentelijke Basisgegevens Zaken (RGBZ) 2.0](https://www.gemmaonline.nl/index.php/RGBZ_2.0_in_ontwikkeling)
