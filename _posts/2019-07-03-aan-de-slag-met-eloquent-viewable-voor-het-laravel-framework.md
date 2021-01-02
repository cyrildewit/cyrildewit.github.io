---
title: Aan de slag met Eloquent Viewable voor het Laravel framework
---

*In dit artikel laat ik je zien hoe je aan de slag kunt gaan met Eloquent Viewable. Met dit pakket breng je eenvoudig de benodigde functionaliteiten naar je Laravel applicatie toe om paginaweergaven van Eloquent models bij te houden.*

Na het installeren van dit pakket heb je via een globale functie of facade toegang tot allerlei praktische functies. Het is bijvoorbeeld mogelijk om het aantal weergave te tellen per model of model type. Dit kun je tevens nog filteren op basis van ’uniekheid’, periode of collectie. Ook kun je een soort van wachttijd tussen de paginaweergaven van de bezoeker instellen, zodat er alleen een weergave per gebruikerssessie bijkomt in plaats van per geladen pagina. Verder zijn er nog tal van andere handige mogelijkheden beschikbaar.

Al deze mogelijkheden zullen aan bod komen in deze reeks artikelen over Eloquent Viewable. In dit deel installeren we eerst het pakket installeren en bereiden we je Eloquent models voor. Vervolgens gaan we de paginaweergaven van een model vastleggen en zullen we het aantal paginaweergaven van een Eloquent model ophalen. Ten slotte sorteren we onze query op het aantal weergaven met behulp van een query scope.

## Installatie en Setup

Om dit pakket te kunnen gebruiken dien je het eerst te installeren. Open je opdrachtprompt en navigeer naar het gewenste Laravel-project. Voer vervolgens het volgende commando uit om dit pakket te installeren via Composer.

```bash
composer require cyrildewit/eloquent-viewable
```

> Houd er rekening mee dat dit pakket een aantal vereisten heeft. Deze kun je vinden in de [README](https://github.com/cyrildewit/eloquent-viewable#requirements).

De serviceprovider wordt automatisch geregistreerd door de zogenaamde ‘package discovery’, oftewel pakketdetectie.

Als je geen automatische pakketdetectie gebruikt, moet je de serviceprovider handmatig toevoegen aan je configuratie.

```php
CyrildeWit\EloquentViewable\EloquentViewableServiceProvider::class,
```

### Migraties publiceren

Nadat je het pakket hebt geïnstalleerd en geregistreerd moet je het volgende commando uitvoeren om de meegeleverde database migraties van het pakket te publiceren.

```bash
php artisan vendor:publish --provider="CyrildeWit\EloquentViewable\EloquentViewableServiceProvider" --tag="migrations"
```

Als optionele stap kun je ook het configuratiebestand publiceren. Mijn advies is om dit eerst eventjes te doorzoeken voordat je naar de volgende stap gaat.

```bash
php artisan vendor:publish --provider="CyrildeWit\EloquentViewable\EloquentViewableServiceProvider" --tag="config"
```

Voer nu de migraties uit met het volgende commando:

```bash
php artisan migrate
```

## Eloquent-models klaarmaken

Om de paginaweergaven van een Eloquent model bij te houden moet het model de volgende interface en trait implementeren.

- Interface: `use CyrildeWit\EloquentViewable\Contracts\Viewable;`
- Trait: `use CyrildeWit\EloquentViewable\InteractsWithViews`;

Voorbeeld:

```php
use Illuminate\Database\Eloquent\Model;
use CyrildeWit\EloquentViewable\InteractsWithViews;
use CyrildeWit\EloquentViewable\Contracts\Viewable;

class Post extends Model implements Viewable
{
    use InteractsWithViews;

    // ...
}
```

De `InteractsWithViews` trait voegt de volgende methodes toe aan je model:

- `views()` - Many To Many (Polymorphic) relatie met de weergaven tabel
- `orderByViews` - Query scope die de resultaten sorteert op het aantal weergaven
- `orderByUniqueViews` - Query scope die de resultaten sorteert op het aantal **unieke** weergaven
- `withViewsCount` - Query scope die de views count toevoegd aan de query

## Paginaweergaven vastleggen

Om een paginaweergave vast te leggen, kun je de `record()` methode aanroepen.

```php
views($post)->record();
```

Achter de schermen wordt er eerst gekeken of we deze weergave wel willen vastleggen. Dit hangt namelijk af van een aantal zaken:

- Is de bezoeker een bot?
- Heeft de bezoeker een DNT header, oftewel een Do Not Track header?
- Staat het IP-adres van de bezoeker op de zwarte lijst (in de configuratie)?
- Geldt er nog een wachttijd?

In de configuratie kun je aangeven of je deze voorwaardes wilt meenemen.

In je applicatie zou je dit bijvorbeeld in een controller plaatsen.

```php
// PostController.php
public function show(Post $post)
{
    views($post)->record();

    return view('posts.show', compact('post'));
}
```

## Paginaweergaven ophalen van een model

Net heb je geleerd hoe je paginaweergaven kunt vastleggen, maar hoe haal je ze nu op?

Net zoals het vastleggen, is het ophalen van de weergaven heel eenvoudig.

```php
views($post)->count();
```

Het teruggekeerde getal is het totale aantal weergaven van de model.

Als je alleen het aantal unieke weergaven wilt ophalen, dan kun je `unique()` toevoegen aan de methode keten (method chain).

```php
views($post)
    ->unique()
    ->count();
```

Dit is mogelijk omdat we een unieke ID opslaan bij de opname van de weergave. Deze ID is opgeslagen in een cookie op de computer van de bezoeker.

Waar je rekening moet houden is dat bezoekers deze cookie kunnen verwijderen. Heel nauwkerig is het dus niet.

## Eloquent-models sorteren op aantal weergaven

Een nuttig voordeel van het bijhouden van de paginaweergaven is dat je de models kunt sorteren op 'populariteit'. Hoe meer weergaven een model heeft hoe mee interesse bezoekers hier in hebben. Althans, dat zou je kunnen stellen.

_Tijdens het klaarmaken van je model heb je de `Viewbable` trait geïmplementeerd waardoor je nu de `orderByViews` query scope kunt gebruiken._

Om de models te sorteren op de meeste weergaven, kun je het volgende doen:

```php
use App\Post;

Post::orderByViews()->get();
```

En het sorteren op de minste weergaven ziet er zo uit:

```php
Post::orderByViews('asc')->get();
```

### Sorteren op unieke weergaven

Om de models te ook op 'uniekheid' te sorten, is er een aparte query scope beschikbaar gesteld via de `Viewable` trait. Namelijk de `orderByUniqueViews`.

Deze methode accepteerd dezelfde argumenten in dezelfde volgorde als de `orderByViews` query scope.

De volgende code kun je gebruiken om de models te sorteren op de meeste/minste unieke weergaven.

```php
Post::orderByUniqueViews()->get(); // van hoog naar laag
Post::orderByUniqueViews('asc')->get(); // van laag naar hoog
```

### Sorteren op weergaven tijdens een bepaalde periode

Een andere interessante mogelijkheid die je hebt is dat kunt aangeven in welke periode de weergaven hebben moeten plaatsvinden.

Beide query scopes accepteren als tweede argument een `CyrildeWit\EloquentViewable\Support\Period` instance.

De API van deze klasse ziet er als volgt uit:

#### Tussen twee tijdstippen

```php
$startDateTime = Carbon::createFromDate(2017, 4, 12);
$endDateTime = '2017-06-12';

Period::create($startDateTime, $endDateTime);
```

#### Vanaf een tijdstip

```php
Period::since(Carbon::create(2017));
```

#### Tot en met een tijdstip

```php
Period::upto(Carbon::createFromDate(2018, 6, 1));
```

#### Vanaf 'past'

Soms wil je de periode baseren op de huidige tijd. Bijvoorbeeld: `Carbon::today()->subDays(2)`. Dit is een geldige `Datetime` instance, maar geeft de interne code geen inzicht over de handeling die is verricht. Daarom zijn de volgende methodes beschikbaar om hetzelfde resultaat te behalen.

```php
Period::pastDays(int $days);
Period::pastWeeks(int $weeks);
Period::pastMonths(int $months);
Period::pastYears(int $years);
```

#### Vanaf 'sub'

De voorgaande methodes zijn gebasseerd op `Carbon::today()`. Om ook het tijdstip mee te nemen (Carbon::now()), kun je gebruik maken van de volgende methodes:

```php
Period::subSeconds(int $seconds);
Period::subMinutes(int $minutes);
Period::subHours(int $hours);
Period::subDays(int $days);
Period::subWeeks(int $weeks);
Period::subMonths(int $months);
Period::subYears(int $years);
```
