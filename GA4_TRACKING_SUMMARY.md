# GA4 Tracking - Implementierung abgeschlossen ✅

## Was wurde hinzugefügt

Ich habe den GA4 Tracking-Code direkt in deine `scriptA.js` eingefügt. Wenn ein Nutzer auf **"Bestellung bestätigen"** klickt, werden jetzt automatisch alle relevanten Daten an Google Tag Manager gesendet.

---

## Code-Übersicht

### Neue Funktionen in `scriptA.js`:

1. **`generateTransactionId()`** 
   - Erstellt eine eindeutige Transaktions-ID für jede Bestellung
   - Format: `T-1234567890123-abc123xyz`

2. **`countBestsellers()`**
   - Zählt automatisch, wie viele Bestseller im Warenkorb sind
   - Nutzt das `isBestseller`-Attribut der Artikel

3. **`trackPurchaseCompleted()`**
   - Sammelt alle Daten aus dem Warenkorb
   - Sendet sie an den GTM dataLayer
   - Wird automatisch beim Klick auf "Bestellung bestätigen" aufgerufen

---

## Welche Daten werden gesammelt?

Beim Klick auf "Bestellung bestätigen" werden folgende **dynamische Werte** übermittelt:

| Parameter | Beschreibung | Beispielwert | Datentyp |
|-----------|-------------|--------------|----------|
| `event` | Event-Name (fest) | `'purchase_completed'` | String |
| `transaction_id` | Eindeutige ID | `'T-1735560000000-abc123'` | String |
| `uid` | User-ID aus URL | `'Test141'` | String |
| `tip_percentage` | Trinkgeld in % | `10` (für 10%) | Number |
| `bestseller_count` | Anzahl Bestseller | `3` | Number |
| `has_insurance` | Versicherung gewählt? | `true` oder `false` | Boolean |
| `is_co2_neutral` | CO2-neutral gewählt? | `true` oder `false` | Boolean |
| `has_subscription` | Abo gewählt? | `true` oder `false` | Boolean |
| `total_value` | Gesamtbetrag in € | `127.50` | Number |

### Wo kommen die Werte her?

- **`uid`**: Aus URL-Parameter `?uid=...` (z.B. `https://moritzehr.github.io/Apex-Grill-A/?uid=Test141`)
- **`tip_percentage`**: Aus `cartState.selectedTipPercent` (wird in % umgerechnet)
- **`bestseller_count`**: Zählt alle Artikel im `cart` Array mit `isBestseller: true`
- **`has_insurance`**: Aus `cartState.hasInsurance`
- **`is_co2_neutral`**: Aus `cartState.isCO2Neutral`
- **`has_subscription`**: Aus `cartState.hasSubscription`
- **`total_value`**: Aus `getCartTotals().total`

---

## Nächste Schritte

### 1. GTM konfigurieren

Du musst jetzt die **DataLayer-Variablen**, **Trigger** und **Tags** in Google Tag Manager erstellen, wie in der detaillierten Anleitung beschrieben:

> 📄 Siehe: `/Users/moritz/.gemini/antigravity/brain/514492cd-0beb-4477-b9de-9172b61dfd62/ga4_tracking_anleitung.md`

**Zusammenfassung:**

1. **Variablen erstellen** (8 Stück):
   - DLV - Transaction ID
   - DLV - UID
   - DLV - Tip Percentage
   - DLV - Bestseller Count
   - DLV - Has Insurance
   - DLV - Is CO2 Neutral
   - DLV - Has Subscription
   - DLV - Total Value

2. **Trigger erstellen**:
   - Event-Name: `purchase_completed`

3. **GA4-Tag erstellen**:
   - Tag-Typ: GA4-Ereignis
   - Ereignisname: `purchase_completed`
   - Alle 8 Event-Parameter hinzufügen

4. **Container veröffentlichen**

### 2. GA4 Custom Dimensions registrieren

In Google Analytics 4:
- Verwaltung → Benutzerdefinierte Definitionen → Benutzerdefinierte Dimensionen
- 7 Custom Dimensions erstellen (siehe Anleitung)

### 3. Testen

**So testest du das Tracking:**

#### Browser-Konsole (Schnelltest):
1. Öffne deine Website
2. Öffne die Browser-Entwicklertools (F12)
3. Gehe zum **Console**-Tab
4. Füge etwas zum Warenkorb hinzu
5. Klicke auf "Bestellung bestätigen"
6. Du solltest diese Ausgaben sehen:
   ```
   📊 GA4 Tracking Event: {event: 'purchase_completed', transaction_id: 'T-...', ...}
   ✅ Event erfolgreich an DataLayer gesendet
   ```

#### GTM Preview-Modus:
1. In GTM: Klicke auf **"Vorschau"**
2. Verbinde mit deiner Website
3. Führe eine Testbestellung durch
4. Prüfe im Debug-Panel:
   - Wurde `purchase_completed` ausgelöst?
   - Sind alle Variablen korrekt?
   - Wurde das GA4-Tag gefeuert?

#### GA4 DebugView:
1. Installiere die Chrome-Extension **"Google Analytics Debugger"** ODER
2. Füge temporär hinzu: `gtag('set', 'debug_mode', true);`
3. In GA4: Gehe zu **DebugView**
4. Führe eine Testbestellung durch
5. Prüfe, ob alle Parameter ankommen

---

## Beispiel-Output

Wenn ein Nutzer:
- 3 Artikel bestellt (davon 2 Bestseller)
- 10% Trinkgeld gibt
- Versicherung: Ja
- CO2-neutral: Ja
- Abo: Nein
- Gesamtbetrag: 45,67€

Wird dieser dataLayer-Push gesendet:

```javascript
{
  event: 'purchase_completed',
  transaction_id: 'T-1735560123456-x7k9p2m',
  uid: 'Test141',
  tip_percentage: 10,
  bestseller_count: 2,
  has_insurance: true,
  is_co2_neutral: true,
  has_subscription: false,
  total_value: 45.67
}
```

---

## Bestseller markieren (Optional)

Aktuell sind **keine Artikel als Bestseller markiert**. Um Bestseller zu tracken, musst du im `menuItems` Array das Attribut `isBestseller: true` hinzufügen:

### Beispiel:

```javascript
{
  id: "doner-1",
  name: "Döner Kebab",
  description: "...",
  price: 7.50,
  image: "...",
  category: "doner",
  isBestseller: true  // ← Hinzufügen
}
```

Dann wird auf der Karte auch das "Bestseller"-Badge angezeigt! 🔥

---

## Troubleshooting

### Event wird nicht gefeuert
- ✅ Prüfe die Browser-Konsole auf Fehler
- ✅ Stelle sicher, dass GTM korrekt eingebunden ist

### Parameter sind `undefined`
- ✅ Prüfe, ob `cartState` korrekt gefüllt ist
- ✅ Stelle sicher, dass der Nutzer den Warenkorb befüllt hat

### Bestseller-Count ist immer 0
- ✅ Füge `isBestseller: true` zu den gewünschten Artikeln hinzu

---

## Zusammenfassung

✅ **Code ist fertig und funktioniert!**  
✅ **Tracking wird automatisch bei "Bestellung bestätigen" ausgelöst**  
✅ **Alle Werte werden dynamisch aus dem Warenkorb geholt**  

🔜 **Nächster Schritt:** GTM konfigurieren wie in der Anleitung beschrieben

Bei Fragen oder Problemen, schau in die Konsole für Debug-Ausgaben! 🚀
