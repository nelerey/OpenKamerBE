# Parlementaire Dataset
Deze repository heeft tot doel een reeks hulpmiddelen te bieden voor het verzamelen, parseren en koppelen van gegevens geproduceerd door het Belgische Federale Parlement (dekamer.be). 
Een gedetailleerde toelichting van de types documenten is hier terug te vinden: https://www.dekamer.be/kvvcr/showpage.cfm?section=/searchlist&language=nl&html=/site/wwwroot/searchlist/typedocN.html#item0-overview

Het project neemt veel inspiratie van [openkamer](https://www.openkamer.org/) onze nederlands tegenhanger, maar we hebben geen API voor de kamer van Belgie. Deze repo is dus een eerste aanzet tot het maken van de scrapers, parsers en scripts nodig om dezelfde info uit de ruwe rapportering van de kamer te halen.

# Very WIP
Deze hele repo is very WIP en is een collectie losse scriptjes die onderzoekend en exploratief zijn bedoeld om te bewijzen dat de data te parsen is. Een volgende stap zet alles dat hierdoor geleerd wordt samen tot een mooier geheel

# Technisch
Alles werkt met Python. Package management wordt gedaan door [PDM](https://pdm-project.org/en/latest/). Lees de documentatie van PDM over hoe je packages moet toevoegen/verwijderen etc. https://pdm-project.org/latest/usage/dependency/

```
git clone git@github.com:thepycoder/OpenKamerBE.git
cd OpenKamerBE
pdm install
```

De TL:DR; van PDM is:
1. `pyproject.toml` is bascially de requirements.txt equivalent, de `pdm.lock` file is de "uitgerekende" lijst van packages en dependencies die moeten gesinstalleerd worden. De flow is meestal: `pyproject.toml` -> `pdm.lock` -> environment
2. "Uitgerekend" betekend hier: zien dat alle versies van alle packages en hun dependiencies samen kunnen bestaan. bvb pkg1: pandas>2 en pkg2: pandas<2 zou nie gaan.
3. Command flow:
- Een nieuwe package toevoegen:
```pdm add <package>```
- Een package verwijderen:
```pdm remove <package>```
- Je lokale environment updaten met een nieuwe (via git most likely) `pdm.lock` file
```pdm sync```
- Nadat je bvb handmatig iets in `pyproject.toml` hebt toegevoegd, opnieuw uitrekenen naar nieuwe lockfile
```pdm lock```


# Data Loaders
## Parlementairen: De mensen van het parlement
Er is een scraper die informatie over alle parlementariers scraped via deze link: https://www.dekamer.be/kvvcr/showpage.cfm?section=/depute&language=nl&cfm=/site/wwwcfm/depute/cvlist54.cfm (alleen deze regeerperiode, andere periodes zijn todo)

Er zijn altijd 150 parlementariers, maar op de webpagina staan er 188 omdat er soms mensen stoppen, gewisseld worden etc.

De code hiervoor staat onder `mensen`. `get_html.py` is made to download the raw html, which lowers the load the dekamer.be while doing frequent iterations. It can take a while because an IP can easily be blocked. `get_people.py` then runs on the htmls files and extracts the people info into JSON.

```
cd mensen
python get_html.py
python get_people.py
```

## Parlementaire Stukken
Parlementaire stukken zijn alle documenten die het parlement produceert: https://www.dekamer.be/kvvcr/showpage.cfm?section=/flwb&language=nl&cfm=Listdocument.cfm?legislat=55
Dit gaat over wetsontwerpen, wetsvoorstellen, verslagen, voorstellen van resolutie etc.

Elk van deze documenten heeft een `fiche`.
Bvb: 
document: https://www.dekamer.be/kvvcr/showpage.cfm?section=/flwb&language=nl&cfm=/site/wwwcfm/flwb/flwbn.cfm?lang=N&legislat=55&dossierID=3698
fiche: https://www.dekamer.be/kvvcr/showpage.cfm?section=flwb&language=nl&cfm=/site/wwwcfm/search/fiche.cfm?ID=55K3698&db=FLWB&legislat=55

`read_fiche.py` is gemaakt om deze fiches te parsen naar JSON voor elk document binnen de regeerperiode.


## Plenaire Vergadering
In de plenaire vergadering wordt op deze stukken gestemd indien het zover is. Dit is mogelijks de belangrijkste moment: het bepaalt welke wetgeving in actie treedt.
De verslagen van deze vergadering staan appart: https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=plen&cricra=CRI&count=all&legislat=55

```
cd plenaire
python get_html.py
python get_plenaire.py
```

## Common
In het mapje `common` staat code die bedoeld is gebruikt te worden door alle losse scripts. Het bepaalt hoe we data gaan opslaan in de backend.
Specifiek zijn hier al een aantal regels, wanneer deze niet voldaan worden voor een nieuw script, moet dat aangepast worden en idealiter in deze folder terecht komen.

### Regels
Namen:
- Volgorde: Achternaam Voornaam
- Accenten en speciale tekens worden gestript met [unidecode](https://pypi.org/project/Unidecode/)
- Gebruik of extend `fix_name` functie in `common`

Partijen:
- Partijen specifiek genaamd in Brussel (e.g. PTB*PVDA enkel voor Brussel, anders PTB of PVDA appart) wordt gerekend tot de waalse partij


## Funny
Er is een mapje `funny` waar je screenshots van grappige passages in kan droppen


# Eurovoc
https://op.europa.eu/en/web/eu-vocabularies/concept-scheme/-/resource?uri=http://eurovoc.europa.eu/100141