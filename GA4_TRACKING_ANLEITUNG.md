# GA4 Button-Klick Tracking - Detaillierte Anleitung

## Übersicht
Diese Anleitung zeigt dir, wie du Button-Klicks in deinem Webshop trackst und dabei folgende Parameter an Google Analytics 4 übermittelst:
- `transaction_id`
- `uid` (User-ID aus URL)
- `tip_percentage`
- `bestseller_count`
- `has_insurance`
- `is_co2_neutral`
- `has_subscription`
- `total_value`

---

## Methode 1: Google Tag Manager (Empfohlen)

### Schritt 1: DataLayer-Event im Code implementieren

Füge folgenden Code zu deinem Button-Klick-Event hinzu:

```javascript
// Beispiel für einen Button-Klick (z.B. "Bestellung abschließen")
document.getElementById('checkout-button').addEventListener('click', function() {
  // Daten für das Tracking
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    'event': 'purchase_completed',  // Event-Name - kannst du anpassen
    'transaction_id': 'T-12345',     // Deine Transaktions-ID
    'uid': 'Test141',                // User-ID aus URL-Parameter
    'tip_percentage': 10,            // Trinkgeld in Prozent
    'bestseller_count': 3,           // Anzahl der Bestseller im Warenkorb
    'has_insurance': true,           // Boolean: Hat Versicherung?
    'is_co2_neutral': false,         // Boolean: CO2-neutral?
    'has_subscription': true,        // Boolean: Hat Abo?
    'total_value': 127.50           // Gesamtwert der Bestellung
  });
});
```

#### Dynamische Werte einsetzen

In der Praxis holst du die Werte aus deinem Shop-System:

```javascript
function trackPurchase() {
  // Hole die Werte aus deinem Shop
  const transactionId = generateTransactionId(); // Deine Funktion
  const uid = getUrlParameter('uid'); // Hole uid aus URL
  const tipPercentage = calculateTipPercentage(); // Deine Funktion
  const bestsellerCount = countBestsellers(); // Deine Funktion
  const hasInsurance = checkInsurance(); // Deine Funktion
  const isCO2Neutral = checkCO2Neutral(); // Deine Funktion
  const hasSubscription = checkSubscription(); // Deine Funktion
  const totalValue = calculateTotalValue(); // Deine Funktion

  // Sende an DataLayer
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    'event': 'purchase_completed',
    'transaction_id': transactionId,
    'uid': uid,
    'tip_percentage': tipPercentage,
    'bestseller_count': bestsellerCount,
    'has_insurance': hasInsurance,
    'is_co2_neutral': isCO2Neutral,
    'has_subscription': hasSubscription,
    'total_value': totalValue
  });
}

// Event-Listener hinzufügen
document.getElementById('checkout-button').addEventListener('click', trackPurchase);
```

---

### Schritt 2: Google Tag Manager konfigurieren

#### 2.1 DataLayer-Variablen erstellen

1. **Öffne GTM** → Gehe zu deinem Container
2. **Klicke auf "Variablen"** im linken Menü
3. **Unter "Benutzerdefinierte Variablen"** → Klicke auf "Neu"
4. **Erstelle für jeden Parameter eine Variable:**

| Variable Name | Typ | DataLayer-Variablenname |
|--------------|-----|------------------------|
| DLV - Transaction ID | Datenschichtvariable | transaction_id |
| DLV - UID | Datenschichtvariable | uid |
| DLV - Tip Percentage | Datenschichtvariable | tip_percentage |
| DLV - Bestseller Count | Datenschichtvariable | bestseller_count |
| DLV - Has Insurance | Datenschichtvariable | has_insurance |
| DLV - Is CO2 Neutral | Datenschichtvariable | is_co2_neutral |
| DLV - Has Subscription | Datenschichtvariable | has_subscription |
| DLV - Total Value | Datenschichtvariable | total_value |

**Schritte für jede Variable:**
- Variablentyp: **Datenschichtvariable**
- Name der Datenschichtvariablen: z.B. `transaction_id`
- Version der Datenschicht: **Version 2**
- Speichern

#### 2.2 Trigger erstellen

1. **Klicke auf "Trigger"** → "Neu"
2. **Trigger-Name:** `Custom Event - Purchase Completed`
3. **Trigger-Typ:** `Benutzerdefiniertes Ereignis`
4. **Ereignisname:** `purchase_completed` (muss mit dem Event-Namen im Code übereinstimmen!)
5. **Speichern**

#### 2.3 GA4-Tag erstellen

1. **Klicke auf "Tags"** → "Neu"
2. **Tag-Name:** `GA4 - Purchase Completed Event`
3. **Tag-Typ:** `Google Analytics: GA4-Ereignis`
4. **Konfiguration:**
   - **Mess-ID oder Konfigurationstag:** Wähle dein GA4-Konfigurationstag aus (oder gib deine Measurement-ID ein: G-XXXXXXXXXX)
   - **Ereignisname:** `purchase_completed`
   
5. **Ereignisparameter hinzufügen:**

   Klicke auf "Ereignisparameter" → Füge folgende Parameter hinzu:

   | Parametername | Wert (Variable) |
   |--------------|-----------------|
   | transaction_id | {{DLV - Transaction ID}} |
   | uid | {{DLV - UID}} |
   | tip_percentage | {{DLV - Tip Percentage}} |
   | bestseller_count | {{DLV - Bestseller Count}} |
   | has_insurance | {{DLV - Has Insurance}} |
   | is_co2_neutral | {{DLV - Is CO2 Neutral}} |
   | has_subscription | {{DLV - Has Subscription}} |
   | value | {{DLV - Total Value}} |

   > [!IMPORTANT]
   > Der `value`-Parameter ist ein Standard-Parameter in GA4 und sollte für den Gesamtwert verwendet werden.

6. **Trigger auswählen:**
   - Wähle den zuvor erstellten Trigger `Custom Event - Purchase Completed`

7. **Speichern**

#### 2.4 Container veröffentlichen

1. **Klicke oben rechts auf "Senden"**
2. **Versionsname:** z.B. `Purchase Tracking mit Custom Parameters`
3. **Versionsbeschreibung:** Beschreibe die Änderungen
4. **Veröffentlichen**

---

### Schritt 3: In GA4 Custom Dimensions registrieren

Damit die Parameter in GA4-Berichten erscheinen, musst du sie als Custom Dimensions registrieren:

1. **Öffne Google Analytics 4**
2. **Klicke auf "Verwaltung"** (Zahnrad unten links)
3. **In der Property-Spalte**: **Benutzerdefinierte Definitionen** → **Benutzerdefinierte Dimensionen**
4. **Klicke auf "Benutzerdefinierte Dimension erstellen"**

Erstelle für jeden Parameter eine Dimension:

| Dimensionsname | Bereich | Ereignisparameter |
|----------------|---------|-------------------|
| Transaction ID | Ereignis | transaction_id |
| User ID (UID) | Ereignis | uid |
| Tip Percentage | Ereignis | tip_percentage |
| Bestseller Count | Ereignis | bestseller_count |
| Has Insurance | Ereignis | has_insurance |
| Is CO2 Neutral | Ereignis | is_co2_neutral |
| Has Subscription | Ereignis | has_subscription |

> [!NOTE]
> Der `value`-Parameter benötigt keine Custom Dimension, da er bereits eine Standard-Metrik in GA4 ist.

**Für jede Dimension:**
- **Dimensionsname:** Sprechender Name (wird in Berichten angezeigt)
- **Bereich:** `Ereignis`
- **Ereignisparameter:** Der genaue Name aus dem GTM-Tag
- **Speichern**

> [!WARNING]
> Custom Dimensions können bis zu 24-48 Stunden benötigen, bis sie in Berichten verfügbar sind!

---

## Methode 2: Direktes gtag.js (Ohne GTM)

Falls du GTM nicht verwendest, kannst du direkt mit `gtag.js` arbeiten:

```javascript
// Stelle sicher, dass gtag initialisiert ist
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}

// Button-Klick tracken
document.getElementById('checkout-button').addEventListener('click', function() {
  gtag('event', 'purchase_completed', {
    'transaction_id': 'T-12345',
    'uid': new URLSearchParams(window.location.search).get('uid'),
    'tip_percentage': 10,
    'bestseller_count': 3,
    'has_insurance': true,
    'is_co2_neutral': false,
    'has_subscription': true,
    'value': 127.50
  });
});
```

Auch hier musst du die Custom Dimensions in GA4 registrieren (siehe Schritt 3 oben).

---

## Testing & Debugging

### GTM Preview-Modus

1. **In GTM:** Klicke oben rechts auf **"Vorschau"**
2. **Gib deine Website-URL ein** und klicke auf "Connect"
3. **Führe den Button-Klick aus**
4. **Im GTM-Debug-Panel:**
   - Überprüfe, ob das Event `purchase_completed` ausgelöst wurde
   - Prüfe, ob alle DataLayer-Variablen korrekt gesetzt sind
   - Prüfe, ob das GA4-Tag gefeuert wurde

### GA4 DebugView

1. **Aktiviere Debug-Modus:**
   ```javascript
   // Füge dies temporär zu deiner Seite hinzu
   gtag('set', 'debug_mode', true);
   ```
   
   **ODER** verwende die Chrome-Extension **"Google Analytics Debugger"**

2. **In GA4:** Gehe zu **"Verwalten" → "DebugView"**
3. **Führe den Button-Klick aus**
4. **Überprüfe:**
   - Ob das Event `purchase_completed` angezeigt wird
   - Ob alle Parameter mit korrekten Werten vorhanden sind

### Browser-Konsole

Teste den DataLayer vor der GTM-Konfiguration:

```javascript
// Führe in der Browser-Konsole aus, nachdem du auf den Button geklickt hast:
console.log(window.dataLayer);

// Du solltest ein Objekt mit deinem Event sehen:
// {event: 'purchase_completed', transaction_id: 'T-12345', ...}
```

---

## Best Practices

### 1. Transaction ID
```javascript
// Generiere eine eindeutige ID pro Transaktion
function generateTransactionId() {
  return 'T-' + Date.now() + '-' + Math.random().toString(36).substr(2, 9);
}
```

### 2. Datenschutz beachten
> [!CAUTION]
> Stelle sicher, dass du keine personenbezogenen Daten (PII) wie Namen, E-Mail-Adressen oder Adressen direkt in GA4 sendest! Dies verstößt gegen die DSGVO.

### 3. Fehlerbehandlung
```javascript
function trackPurchase() {
  try {
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({
      'event': 'purchase_completed',
      'transaction_id': getTransactionId(),
      // ... weitere Parameter
    });
  } catch (error) {
    console.error('Fehler beim Tracking:', error);
  }
}
```

### 4. Nur bei erfolgreichem Checkout tracken
```javascript
// Tracke nur, wenn die Bestellung wirklich erfolgreich war
async function completeCheckout() {
  try {
    const response = await processPayment();
    if (response.success) {
      // Nur bei Erfolg tracken
      trackPurchase();
    }
  } catch (error) {
    console.error('Checkout-Fehler:', error);
  }
}
```

---

## Checkliste

- [ ] Code-Implementierung: DataLayer-Push beim Button-Klick
- [ ] GTM: DataLayer-Variablen erstellt (8 Stück)
- [ ] GTM: Custom Event Trigger erstellt
- [ ] GTM: GA4-Event-Tag mit allen Parametern erstellt
- [ ] GTM: Container veröffentlicht
- [ ] GA4: Custom Dimensions registriert (7 Stück)
- [ ] Testing: GTM Preview-Modus getestet
- [ ] Testing: GA4 DebugView überprüft
- [ ] Testing: In GA4-Berichten nach 24-48h überprüft

---

## Troubleshooting

### Event wird nicht in GTM gefeuert
- ✅ Überprüfe den Event-Namen im Code (`purchase_completed`)
- ✅ Überprüfe den Event-Namen im GTM-Trigger
- ✅ Stelle sicher, dass der Code ausgeführt wird (Console-Log hinzufügen)

### Parameter kommen nicht in GA4 an
- ✅ Überprüfe die Parameternamen in GTM (müssen exakt übereinstimmen)
- ✅ Überprüfe, ob die Custom Dimensions in GA4 erstellt wurden
- ✅ Warte 24-48h, da neue Dimensionen Zeit brauchen

### DataLayer ist undefined
- ✅ Stelle sicher, dass GTM korrekt eingebunden ist
- ✅ Initialisiere den DataLayer: `window.dataLayer = window.dataLayer || [];`

---

## Zusammenfassung

Mit dieser Anleitung kannst du vollständiges Tracking für deine Button-Klicks implementieren. Die empfohlene Methode ist **GTM**, da sie flexibler ist und Änderungen ohne Code-Deployment ermöglicht.

**Wichtigste Schritte:**
1. DataLayer-Push im Code implementieren
2. GTM konfigurieren (Variablen, Trigger, Tag)
3. Custom Dimensions in GA4 registrieren
4. Ausgiebig testen

Viel Erfolg! 🚀
