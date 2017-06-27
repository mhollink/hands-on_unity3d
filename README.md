# Unity3D : Hands-on 27 Juni 2017

## Requirments

* C# editor

    * [Visual Studio](https://www.visualstudio.com/) (Windows only)
    * [Visual Studio Code](https://code.visualstudio.com/) (Any OS)
    * [MonoDevelop](http://www.monodevelop.com/) (Mee geleverd met Unity3D, misschien buggy)
	* [Rider *EAP](https://www.jetbrains.com/rider/) (Any OS, Early access voor C# editor, IntelliJ style)
    * Notepad (If everything fails)
    
* Unity3D

    * [Windows/Mac](https://unity3d.com/)
    * [Linux](http://beta.unity3d.com/download/45784aaa9968/public_download.html)

In deze handson sessie gaan we een race game ontwikkelen waarbij er een AI gestuurd voertuig over een baan rijd. Daarnaast gaan we kijken naar input van een speler om er voor te zorgen dat jij zelf met een andere auto over de baan kan rijden. In dit git project vind je alle assets die nodig zijn voor de ontwikkeling van de game. 

# Stap 0: Open het project in Unity. 

* Open vervolgens de Scene in de map 'Scenes'.

# Stap 1: Maak een path dat door de auto gevolgd kan worden.

Om een te maken moeten er waypoints worden toegevoegd aan de wereld. Hiervoor gebruiken we het transform component. Elk transform component bevat de x-y-z positie van een waypoint.

## Aanmaken van objecten

1. Maak een nieuwe 'empty' in de speel wereld en noem deze 'Path'. Dit wordt het parent object voor alle waypoints.

2. Maak een nieuwe 'empty' in de parent en noem deze 'Waypoint'.

3. Kopier dit Waypoint tot je er minimaal 3 hebt (ctrl+d).

4. Voeg een nieuw script component toe aan het 'Path' object en geef deze een soortgelijke naam en open het script in de IDE.

  * Er kan gekozen worden uit C# en JavaScript, Kies voor C# (default)

  * In het script bevinden zich ene Start en Update method. Deze zijn niet nodig en kunnen dus worden verwijderd.

      
      
Om er voor te zorgen dat we zien hoe het path gevolgd wordt gaan we gebruik maken van de UnityEditor functies. De UnityEditor functies worden, zoals de naam sugereerd, uitgevoerd in de Unity Editor. Deze functies worden niet gebruikt tijdens runtime.
      
## Gebruiken van UnityEditor functies
  
1. Voeg de 'OnDrawGizmos' methode toe aan het Path script.

```
void OnDrawGizmos()
{
 
}
```

2. Zoek alle transforms die in het parent object zitten. De methode `GetComponentsInChildren<Transform>()` wordt hier voor gebruikt.

3. De methode uit stap 2 maakt een array van alle transforms binnen het parent object. Dit betekend dat ook het parent object zelf in deze array zit. Om deze er uit te krijgen stoppen we alle child transforms in een List. 

```
List<Transform> waypoints = new List<Transform>();
foreach (Transform transform in transforms)
{
    if (transform != this.transform) // this.transform refereert naar het transform van het MonoBehaviour, wat bij default is overgeerfd. 
        waypoints.Add(transform);
}
```

4. Nu we alle child transforms hebben kunnen we deze zichtbaar maken door een Sphere om de transforms te teken. `Gizmos.DrawWireSphere(positie, radius)` is hiervoor een geschikte methode.

```
for(int i = 0; i < waypoints.Count; i++)
{
    Vector3 currentWaypoint = waypoints[i].position;

    Gizmos.DrawWireSphere(currentWaypoint, 0.3f);
}
```

  * Om het pad zelf zichtbaar te maken kan je gebruik maken van `Gizmos.DrawLine(currentWaypoint, previousWayPoint)`. Het vorige waypoint kan je op een zelfde manier selecteren als het huidige. 
    
* Maak het pad af zodat deze de baan volgd. 
 
* Om er voor te zorgen dat het pad niet meer zichtbaar is tijdens het aanpassen van de rest van de scene kan de naam van de methode worden veranderd naar `OnDrawGizmosSelected()`. Hierdoor worden de waypoints enkel zichtbaar als het Path object geselecteerd is.
  
# Stap 2: Basis functies voor de auto.

In de scene bevind zich al een auto. Deze auto bestaat enkel uit visuele componenten. Om er voor te zorgen dat de auto bruikbaar is in de game moeten hier nog functionaliteit aan worden toegevoegd. We beginnen nu eerst met de physics.

## Physics

1. In Unity3D zit een component met de naam 'Rigidbody'. Dit component wordt gebruikt voor gravitatie. Voeg een Rigidbody toe aan het parent object van de auto.

    * In het Rigidbody vind je verschillende variablen. Aangezien we over een auto spreken is een 'Mass' van 1 niet voldoende. Voor accurate physics moet de mass naar ongeveer 1300. 
    
2. Wanneer de game nu gestart wordt valt het auto object door de grond heen. Dit komt omdat het voertuig wel beinvloed wordt door de zwaartekracht, maar geen collision kan veroorzaken. Hiervoor moet er een collider worden toegevoegd aan de auto. In het child object bevind zich een object met de naam CarClosed, voeg hier een meshcollider aan toe. 

    * De meshcollider maakt gebruik van het model in het object. De zichtbare randen worden gebruikt als colliders. 
    
    * Elke 'face' van de mesh wordt gebruikt voor collision, dit is slecht voor performance. Door convex aan te vinken wordt er een collider gemaakt die 'ongeveer' het hele object bevat. Voor onze game is dit dan ook een must.
    
3. De wielen in ons voertuig moeten nu ook nog een collider. Om de 'Wheel Colliders' goed te laten werken moeten een paar stappen worden voltooid:

    1. Maak een object aan in het 'CarSpecial' object van het voertuig noem het 'Wheels'. Zet zijn positie op 0-0-0, doe het zelfde voor zijn rotatie. Laat de scale op 1-1-1.
    
    2. Voeg alle wielen toe aan het wheels object.
    
    3. Dupliceer het wheels object en noem het 'Wheels (1)' object 'WheelColliders'
    
    4. Selecteer alle wielen in het WheelColliders object, en verwijder alle component op het transform component na. (Klik rechts op het tandwiel van een component)
    
    5. Voeg het 'Wheel Collider' component toe aan de lege wielen.
    
    6. Zorg er voor de radius in het wheel collider overeenkomt met de grote van het wiel. +/- 0.105
    
* Wanneer er nu op play wordt gedruk zou de auto net boven de grond moeten zweven.

Om er voor te zorgen dat onze auto het pad gaat volgend dat eerder is gemaakt moet de auto kunnen sturen. Hiervoor gaan we gebruik maken van verschillende functies in de inspector. We hebben eerder al componenten toegevoegd aan de objecten via de inspector, hier gaan we nu dan ook verder op in.

## Steering

1. Voeg aan de auto een nieuw script toe en noem deze 'Car'. Dit script moet in het zelfde object geplaatst worden als het Rigidbody.

2. Om een referentie te creeeren naar de way points moeten we het Path toe voegen. De eerste regel die we toevoegen aan het object is `public Path path;`

    * Doordat de variablen public is wordt deze zichtbaar in de inspector.

3. Sla het script op en voeg het path object toe aan het script. Dit kan gedaan worden door het object met het 'Path' script er op te slepen.

4. Nu we toegang hebben tot het path object kunnen we de waypoints uit de scene ophalen. Dit kan op een soortgelijke manier gedaan worden als in het 'Path' script.

    * Maak een private field aan om alle waypoints in op te slaan. 

    ```
    private List<Transform> waypoints;
    ```
    
    * In de start methode, voeg alle waypoints toe aan de list.
    
    ```
    foreach (Transform transform in path.transforms)
    {
        if (transform != path.transform) // path.transform refereert naar het transform van het MonoBehaviour in het Path object. 
            waypoints.Add(transform);
    }
    ```
    
5. Nu we alle waypoints hebben kunnen we beginnen aan de logica voor het sturen. Het MonoBehaviour komt standaard met de `void Update()`. Deze update wordt ieder frame aangeroepen. Voor onderdelen van een object die te maken hebben met physics is dit niet gewenst. In plaat daarvan stappen we over op de `void FixedUpdate()`. Verwijderd de `Update` en plaats een `FixedUpdate` in het Car script.

6. Om in de richting van het volgende waypoint te sturen moeten de relatieve locatie van het waypoint weten. Met andere woorden, we zijn opzoek naar de x-y-z afstand tussen het voertuig en het waypoint. Een methode in Unity hiervoor is `InverseTransformPoint`. Plaats de volgende regel code in de fixed update:

```  
Vector3 relativeVector = transform.InverseTransformPoint(waypoints[waypointIndex].position);
```

* Om bij te houden bij welke waypoint we zijn houden we een index bij. Hiervoor kan er een private field worden gemaakt.
    
7. Om de hoek van de wielen te berekenen willen we de relatieve.x locatie hebben van het waypoint, de afstand tussen het voertuig en het waypoint. Daarnaast willen we ook de maximale rotatie van de wielen hebben. De rotatie zullen we vast zetten in een variablen die aan te passen is in de inspector. 

```
[Range(0,90)]
public float maxSteerAngle = 45f;
````

* De range annotation zorgt er voor dat de variablen in de inspector niet buiten die waardes kan vallen
    
* Door de variablen direct een waarde te geven wordt deze als default gebruikt wanneer er niets wordt aangepast.

8. In de FixedUpdate, bereken de steerAngle door de relatieve x afstand te delen door de magnitude. Vermenigvuldig de uitkomst met de `maxSteerAngle`.

9. Nu we de rotatie hebben moeten we deze nog aan de wielen doorgeven. Maak 2 velden voor de WheelColliders. Een voor het linker voorwiel en een voor het rechter voorwiel.

10. In de fixed update an het `steerAngle` worden toegewezen aan beide wielen.

Als alles tot dit punt is goed gegaan zullen de WheelColliders in de richting van het eerste waypoint draaien. De visuele wielen zullen echter niet mee draaien. Om te zien of de WheelColliders draaien moet je deze selecteren in het hierarchy panel.

Nu de wielen de juiste richting in staan gaan we werken aan de verplaatsing. Om alles overzichtelijk te houden kunnen we de code voor het sturen verplaatsen naar een methode. Deze kan je bijvoorbeeld 'steering' noemen. Vergeet niet deze methode aan te roepen in de FixedUpdate.

## Moving

1. Maak een nieuwe methode in de FixedUpdate, direct onder de 'steering' methode. Deze methode gaat de logica bevatten voor de verplaatsing.

2. Het voertuig rechtdoor verplaatsen kan gedaan worden met `motorTorque`. Wanneer deze waarde niet op 0 staat zal het voertuig gaan rijden. Voeg allereerst een aantal variabelen toe.

```
public float MaxMotorTorque = 60f;
public float MaximumSpeed = 120f;

private float currentSpeed;


```

3. Alleer eerst zullen we kijken of het voertuig vooruit gaat.

```
private void Move()
{
    wheelFL.motorTorque = MaxMotorTorque;
    wheelFR.motorTorque = MaxMotorTorque;
}
```

* Wanneer deze code wordt uitgevoerd zal het voertuig naar het waypoint toe rijden. Zodra deze er voorbij is zal hij achteruit rijden. Als het voertuig dichtbij genoeg is, willen we door naar het volgende waypoint.
    
3. Om er voor te zorgen dat de 'waypointIndex' wordt geupdate gaan we kijken naar de afstand tussen het voertuig en het waypoint. Hiervoor heeft unity een functie in het Vector3 object, namelijk distance. Gebruik deze om de te zien of de afstand tussen het voertuig kleiner is dan 0.1.

4. Vul de logica aan om er voor te zorgen dat de index wordt opgeteld.

    * Wanneer het voertuig rondjes om een waypoint blijft rijden betekend dit dat het waypoint te hoog boven de grond zit. 
    
    * Hou er rekening mee dat de index bij aankomst bij het laatste waypoint weer op 0 gezet wordt. 
    
5. Om de snelheid in kilometer per uur te berekenen gebruiken we een formule. deze formule is alsvolgd

```
currentSpeed = ((Mathf.PI*2) * (wheelFL.radius * wheelFL.rpm)) * (60/1000)
```

6. Nu de huidige snelheid bekend is kunnen we deze gebruiken om te kijken we beneden de maximum snelheid zitten. Wanneer dit namelijk niet het geval is willen we de motorTorque op 0 zetten.

```
wheelFL.motorTorque = currentSpeed < maxSpeed ? MaxMotorTorque : 0;
wheelFR.motorTorque = currentSpeed < maxSpeed ? MaxMotorTorque : 0;;
```

De auto zou nu het pad volledig moeten volgen. Daarnaast zal hij nooit boven de aangeven snelheid gaan. Het enige dat nu opvalt is dat de wielen niet mee draaien. Dit is dan ook onze volgende stap.

## Visuele verbetering

1. Maak een nieuw script aan en noem deze 'ColliderToWheel' en voeg een public WheelCollider variabele toe. 

2. Voeg dit script toe aan alle 'Visuele wielen'. Dit zijn de objecten die de mesh bevatten, niet degene met de collider.

3. Voeg voor elk wiel het wheel collider object toe. Let er op dat het linker voorwiel gekoppeld is aan de collider van het linker wiel.

4. Om de positie en rotatie van de wielen te krijgen kunnen we de methode `GetWorldPose` gebruiken. Deze methode vraagt 2 variabele, namelijk een Vector3 (positie) en een Quaternion (rotatie). Deze moeten vooraf geintentieerd worden.

```
private Vector3 wheelPosition = new Vector3();
private Quaternion wheelRotation = new Quaternion();
```

5. In de update functie van dit object gaan we de variabelen updaten. Hiervoor gebruiken we de `GetWorldPose` methode.

```
target.GetWorldPose(out wheelPosition, out wheelRotation);
```

6. Het laatste wat we nu moeten doen is deze variabele gebruiken in het visuele wiel. Aangezien we het script in het wiel hebben gehangen kan de transform direct gebruikt worden.

```
transform.position = wheelPosition;
transform.rotation = wheelRotation;
```

Als je het voertuig nu laat rijden zal zullen alle wielen draaien. Daarnaast zal ook het sturen van de voorwielen zichtbaar zijn. Wanneer we de auto een volledig rondje later rijden zal opgemerkt worden dat hij meerdere malen moeite heeft. Dit komt door het gewicht van het voertuig. Doordat een rigidbody het gewicht altijd in het midden plaats zal het voertuig sneller omrollen. Om dit te voorkomen gaan we het center verplaatsen.

## Gewicht van het voertuig

1. Voeg aan het 'Car' script een nieuwe variabele toe. Deze moet van type Vector3 zijn en zal de naam 'centerOfMass' krijgen.

2. In de start methode moet vervolgens dit trasform gebruikt worden om de center voor het rigidbody te veranderen. Dit kan met de volgende regel code gedaan worden.

```
GetComponents<Rigidbody>().centerOfMass = centerOfMass;
```

3. Zet in de inspector vervolgs deze Vector3 op x:0, y:-.2, z:0

# Stap 3: Artificial intelligence zonder sensoren?  

In de huidige simulatie zal de auto rondjes blijven rijden. Op het moment dat hij dichtbij een waypoint komt zal naar de volgende gaan. Zolang er geen opstacels zijn op de weg te vinden zijn zal dit goed gaan. Wanneer de auto tegen obstacel aanrijd zal er geen besef zijn dat hij niet verder kan. Op dit moment zal de auto dan ook vast staan.

Om er voor te zorgen dat hij om obstacels heen rijd kan er gebruikgemaakt worden van raycasting. Bij raycasting wordt er alvorens de actie gekeken of het mogelijk is. 

## Raycasting

1. Voor de raycasting in onze auto hebben we een aantal variabele nodig. We willen vaststellen hoeveel rays we gebruiken, de lengte en hun start positie. Daarnaast willen we ook weten hoe breed de auto is.

```
[Header("Sensors")]
public int raysCount = 5;
public float raysLength = 2.5f;
public Vector3 raysStartPosition;
public float carWidth = 2f;
```

    * Om onze raycast variabelen te scheiden van de rest maken we gebruik van de header annotatie. Deze zorgt er voor dat de variabele gescheiden worden in de inspector.
    
2. In de fixed update moeten we nu de sensors aanroepen. Hiervoor maken we een methode aan die we als allereerst aanroepen. 

```
void FixedUpdate()
{
    CheckSensors();
    ... // rest van de methods.
}
```

3. Een raycast kan gedaan worden met door gebruik te maken van de Physics class. De raycast methode in Physics heeft 16 verschillende overloads. De overload die gebruikt wordt voor onze auto is de `Physics.Raycast(origin, direction, out raycastHit, distance)`. 

    1. Voor de origin tellen we de positie van de auto op bij de start positie. Aangezien de auto op in de wereld draaid kunnen we deze niet simpel weg optellen doormiddel van `transform.position + raysStartPosition`. In plaats daarvan moeten we de axis op de juiste manier optellen. Dit kan gedaan worden door `transform.forward * raysStartPosition.z` en `transform.up * raysStartPosition.y` op te tellen bij de positie van onze auto.

    2. Voor de richting van de ray gebruiken we de richting waarin de auto rijd. `transform.forward`
    
    3. Voor de raycastHit moeten we een locale variabele maken, deze noemen we bijvoorbeeld 'hit'. `RaycastHit hit`
    
    4. Voor de lengte van de ray kunnen we de variabele raysLength gebruiken.
    
    5. Plaats de Physics.Raycast() in een if-statement
    
        * De Raycast methode geeft true terug wanneer het een collider raakt.
        
* Plaats onder de if-statement de volgende regel code om de raycast zichtbaar te maken `Debug.DrawLine(sensorStartingPosition, sensorStartingPosition + transform.forward, Color.white);` 

Je sensors methode zou er nu ongeveer zoals hieron uit moeten zien

```
RaycastHit hit;
Vector3 sensorStartingPosition = transform.position + 
        (transform.forward * raysStartPosition.z) + 
        (transform.up * raysStartPosition.y);
        
if (Physics.Raycast(sensorStartingPosition, transform.forward, out hit, raysLength))
{
    Debug.DrawLine(sensorStartingPosition, sensorStartingPosition + transform.forward, Color.white);

}
``` 

4. Op dit moment teken we 1 ray. In de variabele hebben we echter genoeg informatie om meerdere rays te tekeken. Hiervoor gebruiken we een for loop om alle rays te tekeken. Als eerste moeten we weten wat de meest linker rand is van de auto. Daarna moeten we de afstand tussen de rays bereken.

```
float rightBound = carWidth / 2f;
float rayDelta = carWidth / (raysCount-1);
```

5. Om de start positie van een elke ray afzonderlijk te bepalen moeten de deze variabelen gebruiken. zoals we eerder transform.forward voor de z axis gebruikt hebben, en transform.up voor de y axis. kunnen we voor de x axis tranform.right gebruiken. Dit vermenigvuldigen we vervolgens met rayDelta * i - rightbound. 

```
transform.right * ((rayDelta * i) - rightBound)
```

Nu de raycast gedaan wordt en het moment waarop een raycast iest raakt bekend is kunnen we hier gedrag aan toekennen. 

## Sensor functionaliteit

1. Maak variabele aan voor de achter wielen.

```
public WheelCollider wheelRL;
public WheelCollider wheelRR;
```

2. Maak een methode aan die er voor gaat zorgen dat de auto kan remmen.

```
private void Break()
{

}
```

3. In de CheckSensors methode moet er een flag gezet worden wanneer een sensor iets raakt. Hiervoor moet er dus een boolean worden aangemaakt. 

4. Zet deze boolean in de op true als een ray iets raakt. Zet deze terug op false als er geen rays zijn die iets raken.

5. In de FixedUpdate, kijk of de de auto aan het remmen is. Bij true moet Break worden aangeroepen en bij false moet Move worden aangeroepen.

6. In de break methode moeten de voorwielen stil gezet worden, daarnaast moeten de remmen op de achterwielen worden geactiveerd

```
wheelFL.motorTorque = 0;
wheelFR.motorTorque = 0;

wheelRL.brakeTorque = 350;
wheelRR.brakeTorque = 350;
```

7. Het laatste wat nog nodig is, is het los laten van de remmen. Dit kan worden gedaan in de move functie. 

* Door nu een tweede auto recht voor de eesrte te plaatsen zal de sensor merken dat er een object voor hem staat. Hierdoor zal het even duren voor dat hij vertrekt. 

    * Een probleem onstaat nu wanneer twee auto's paralel rijden. Dit probleem kan worden verholpen door extra rays diagonaal uit de auto te casten.
    
De uitdaging voor het diagonaal raycasten laat ik bij jullie. Eerst gaan we nog kijken naar user-input. 

# Stap 4: Input

Voor we input gaan verwerken in onze auto gaan we eerst de code een beetje opruimen. 

## Opruimen en scheiden van code

1. Maak een CarAI en CarPlayer script aan. 

    * Maak het Car script abstact
    
    * Laat CarAI en CarPlayer extenden van het Car script.

2. verplaats de volgende variabele naar het CarAI Script

```
public Path path;
private List<Transform> waypoints;
private int waypointIndex;

[Header("Sensors")]
public int raysCount = 5;
public float raysLength = 2.5f;
public Vector3 raysStartPosition;
public float carWidth = 2f;
```

3. Maak `isBreaking` in het car script protected zodat het CarAI script er bij kan.

4. Geef de hieronder genoemde methodes een `protected` access modifier

```
protected void Move()
protected void Break()
```

5. Plaats de overige methodes in het CarAI script. 

6. Maak een private methode aan in het CarAI script en plaats hier het updaten van de WayPointIndex in. 

    * Verwijder dit stuk code uit de Move methode.
    
    * Roep deze methode aan in de FixedUpdate, vlak voor de move functie.

```
private void UpdateWaypointIndex() 
{
    if (Vector3.Distance(transform.position, waypoints[waypointIndex].position) < 1f)
        waypointIndex = waypointIndex < waypoints.Count - 1 ? waypointIndex + 1 : 0;
}
```

Als alles goed is verplaats beschikt het Car.cs script nu over de volgende informatie :

* Maximale stuur hoek
* Alle wiel colliders
* De centerOfMass
* Maximale snelheid
* Huidige snelheid
* Of de auto aan het remmen is
* de Move() methode
* de Break() methode

Kijk in de Unity inspector of de auto het juiste script heeft. Wanneer dit niet het geval is zal deze moeten worden toegevoegd. 

Test of de Car AI nog werkt. Zodra dit het geval is kan er verder worden gegaan met het maken van Input.

## Implementeren van input

Om input van een speler te gebruiken in Unity kan er gebruik gemaakt worden van de Input class. In de Input class bevinden zich meerdere methodes. De methodes die voor ons belangrijk zijn zijn Input.GetAxis() en Input.GetButton(). 

1. Voor dat de door de speler bestuurde auto kan werken moeten we eerst de centerOfMass weer goed zetten. Deze variabele bevind zich reeds in het Car script. We hoeven dus enkel `GetComponent<Rigidbody>().centerOfMass = centerOfMass;` in de start methode te plaatsen.

2. Plaats een update methode in het CarPlayer script.

``` 
void Update()
{

}
```

In de update loop gaan we kijken naar verschillende input van de speler. Deze zijn voor:

* Rijden

* Sturen

* Remmen
    
3. In de InputManager van unity zijn namen gegeven aan verschillende knoppen. Hiervoor kan gekeken worden onder 'Edit > Project Settings > Input'. De input die wij gaan gebruiken zijn Horizontal, Vertical en Jump. Belangrijk hierbij is dat Horizontal en Vertical op een axis liggen. Maak voor alle input variabele aan.

```
private float horizontal;
private float vertical;
private bool breaks; 
```

4. Update de waarde van deze variabele in de Update loop

```
horizontal = Input.GetAxis("Horizontal");
vertical = Input.GetAxis("Vertical");
breaks = Input.GetButton("Jump");
```

5. Maak nu een FixedUpdate voor het aanroepen van het rijden of remmen.

```
void FixedUpdate()
{
    if (breaks) Break();
    else
    {
        if (vertical > 0) Move();
    }
}
```

* Wat gebeurt er nu als je het gas loslaat?

6. Om er voor te zorgen dat de motor stopt en de auto rollend tot stilstand komt moeten we de motorTorque op de voorwielen op 0 zetten. Maak hiervoor een methode aan in het Car script en roep deze aan het het CarPlayer script wanneer vertical kleiner of gelijk is aan 0.

```
protected void ResetMotorTorque()
{
    wheelFL.motorTorque = 0;
    wheelFR.motorTorque = 0;
}
```

7. Voor het naar links en naar rechts sturen maken we gebruik van de horizontal float. Maak een method `ApplySteering()` in de CarPlayer en roep deze boven het remmen aan in de FixedUpdate.

8. In deze methode kan nu de logica geplaatst worden voor het sturen. Hou rekening met de volgende condities

    * horizontal > 0
    * horizontal < 0
    * horizontal == 0
    
9. Nu het sturen is geimplementeerd kan je het voertuig in de scene kopieren. Verwijder het CarAI script en voeg het CarPlayer script toe.

Verwijder de camera in de AI gestuurde auto. Dit zorgt er voor dat de Camera in de bestuurbare auto gebruikt word.

  