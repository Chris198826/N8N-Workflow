## Überblick

Dieser Workflow in **n8n** verknüpft mehrere Dienste und Nodes zu einer komplexen Pipeline für Text- bzw. Sprachverarbeitung. Er kombiniert:

1. **Telegram-Bots** (zwei verschiedene Bots),
2. **OpenAI**-Modelle (ChatGPT/Whisper für Sprach-Transkription und Textgenerierung),
3. **Google Sheets** (als Speicher und „Gedächtnis“ für die Chat-Konversation bzw. KI-Ausgaben) und
4. **Trello**-Listen (für automatisches Anlegen von Karten je nach Keyword).

Zudem werden in den verschiedenen **Sticky-Notes** (Notizzetteln) wichtige Vorarbeiten erläutert, z. B. wie man Konten anlegt und in n8n einbindet. Nachfolgend eine Schritt-für-Schritt-Anleitung, was in diesem Workflow passiert und wie man ihn einrichtet.

---

## 1. Telegram-Bots anlegen und verbinden

1. **Zwei Bots erstellen (optional)**  
   - Nutze den Telegram-Bot [@BotFather](https://t.me/BotFather).  
   - Erstelle dort Bots (beispielsweise **Bot A** und **Bot B**).  
   - Notiere dir die **API-Tokens**, die du von @BotFather bekommst.

2. **Bots in den gewünschten Kanal/Channel einladen**  
   - In Telegram den entsprechenden Kanal (Channel) erstellen oder auswählen.  
   - Füge beide Bots (A und B) als Admin hinzu (oder zumindest mit Schreib- und Leserechten).

3. **In n8n die Telegram-Credentials hinterlegen**  
   - Öffne deine n8n-Instanz, gehe zu **Credentials** → **Telegram**.  
   - Trage dort die Bot-Token ein, die du vom @BotFather erhalten hast.  
   - Im Workflow selbst findest du z. B. **„Telegram Trigger (Main)“**, „Telegram Send #1 (Bot B)“, „Telegram Send #3 (Bot A)“ usw. Achte darauf, dass jede **Telegram Node** die richtigen Credentials verwendet.

4. **Telegram Trigger (Main)**  
   - Dieser Node lauscht auf eingehende Nachrichten (Text oder Voice) in dem Kanal, in dem der Bot als Admin ist.  
   - Sobald eine Nachricht eingeht, wird der Workflow ausgelöst.

---

## 2. Google-Account und Google Sheets

1. **Google-Konto einrichten** (falls noch nicht vorhanden).  
2. **Google Sheets-Vorlage** erstellen bzw. kopieren:  
   - Im Workflow ist ein Verweis auf eine Beispiel-Tabelle (z. B. „Vorlage Speicher Bots“).  
   - Darin sind Spalten wie „Memory“ und „output“ vorgesehen, um KI-Outputs zu speichern.
3. **Credentials in n8n** hinterlegen:  
   - Unter **Credentials** → **Google** (z. B. „Google Service Account“ oder OAuth2)  
   - Stelle sicher, dass deine Google Sheets API für diesen Account freigegeben ist.
4. **Google Sheets Nodes**  
   - Hier werden die KI-Antworten und „Memory“-Daten angehängt bzw. ausgelesen (Nodes wie „Speicher“, „Speicher1“, „Google Sheets11“ usw.).

---

## 3. OpenAI (ChatGPT/Whisper) einrichten

1. **OpenAI-Account** anlegen, falls nicht vorhanden.  
2. **API-Schlüssel** erstellen und in n8n als neue Credentials hinzufügen (unter **Credentials** → **OpenAI**).  
3. **Sprach-Transkription**:  
   - Der Node **„OpenAI“** (Typ „Audio → Transcribe“) nutzt Whisper, um Voice-Nachrichten in Text umzuwandeln.  
   - Achte in den Node-Parametern auf **language** = `de` (für Deutsch).
4. **Chat-Modelle**:  
   - Im Workflow gibt es mehrere OpenAI Chat Model-Nodes (z. B. „OpenAI Chat Model (A1)“, „OpenAI Chat Model (B2)“ usw.).  
   - Sie greifen je nach Prozessschritt auf **GPT-4** oder **GPT-3.5** (bzw. „gpt-4-turbo“, „gpt-4o“) zurück.  
   - Jeder dieser Nodes hat eigene Einstellungen (Temperature, Frequency Penalty etc.).

---

## 4. Trello-Integration

1. **Trello-Account** erstellen und/oder einloggen.  
2. **API-Key und Token** generieren:  
   - Über [Trello Developer](https://developer.atlassian.com/cloud/trello/guides/rest-api/api-introduction/) erhältst du die nötigen Daten.  
   - In n8n unter **Credentials** → **Trello** eintragen.
3. **Listen-ID finden**  
   - Du kannst beispielsweise ein Power-Up benutzen oder in der Trello-URL schauen (die Listen-ID steht in der Adresszeile).  
   - Jede Node im Workflow („Kurse“, „Mail“, „LinkedIn“ etc.) hat eine **listId**, um Karten direkt in die passende Trello-Liste zu erstellen.
4. **Automatische Karten-Erstellung**  
   - Im Workflow geschieht dies über Nodes wie „Kurse“, „Mail“, „Automatisierung“, „Webinare“, „Marketing“, „LinkedIn“.  
   - Je nach erkanntem Keyword wird in der passenden Trello-Liste eine Karte angelegt (mit Beschreibung = KI-Ausgabe).

---

## 5. Aufbau und Ablauf des Workflows in n8n

1. **Telegram Trigger (Main)**  
   - Sobald im Kanal eine neue Nachricht kommt (Text oder Voice), startet der Workflow.

2. **Switch Node „Determine content type1“**  
   - Unterscheidet, ob eine **Text-Nachricht** oder eine **Voice-Nachricht** gesendet wurde.  
   - Falls **Voice**, wird sie an den Node „OpenAI“ (Whisper-Transkription) geschickt.  
   - Falls **Text**, wird direkt weiterverarbeitet.

3. **OpenAI (Sprach-Transkription)**  
   - Nimmt die Audiodatei entgegen und erzeugt daraus Text.  
   - Leitet den transkribierten Text an die KI-Modelle weiter.

4. **AI-Agent-Kette**  
   - Der Workflow hat mehrere KI-Nodes, die in aufeinanderfolgenden Schritten den Text bearbeiten:  
     1. **Agent A1**: Erstellt einen ersten, sehr ausführlichen Rohentwurf basierend auf einem bestimmten Keyword (z. B. „Mail“, „Marketing“ usw.).  
     2. **Agent B-Strategy (A2)**: Optimiert den Text hinsichtlich Struktur, Stil und führt zusätzliche Inhalte ein.  
     3. **Agent B-Creativity (A3)**: Führt ein sprachliches Lektorat durch, veredelt den Text etc.  
     4. **Agent A3 Final merge (A4)**: Finalisiert den Text mit Faktencheck und gibt ihn frei.  
   - Die **„Auto-fixing Output Parser“-Nodes** und „Item List Output Parser“-Nodes dazwischen sorgen dafür, dass die Antworten im gewünschten JSON-Format ausgegeben werden.

5. **Code-Nodes**  
   - Mehrere „Code1“, „Code2“, „Code3“ usw. filtern oder formatieren die KI-Ausgaben. Sie sorgen zum Beispiel dafür, dass:
     - Doppel-„\n“-Zeilenumbrüche entstehen (Markdown-Vorgabe).
     - Die Text-Auszüge ab „### Einleitung“ gekürzt werden.
     - JSON-Strings korrekt geparsed oder bereinigt werden.

6. **Switch-Nodes** (z. B. „Switch Check Choices“, „Switch Next Agent“ usw.)  
   - Sie prüfen, welches **Keyword** in der KI-Ausgabe steckt (z. B. „Mail“, „LinkedIn“, „Automatisierung“, etc.).  
   - Auf Basis dessen wird entschieden, in welche Trello-Liste eine Karte geschrieben wird.

7. **Trello-Nodes** („Mail“, „Automatisierung“, „Webinare“, „Kurse“, „Marketing“, „LinkedIn“)  
   - Legen je nach erkanntem Keyword eine neue Karte an, mit der von der KI erstellten Beschreibung.

8. **Google Sheets-Nodes** („Speicher“, „Speicher1“, „Speicher2“, …)  
   - Speichern den generierten Text oder „Memory“-Infos in der verlinkten Google-Tabelle, sodass die KI bei weiteren Anläufen darauf zugreifen kann (LangChain Memory).

---

## 6. Wichtige vorbereitende Schritte (Zusammenfassung)

- **Telegram**:  
  - Bots via @BotFather erzeugen und in n8n-Credentials eintragen.  
  - In n8n an den entsprechenden Telegram-Nodes referenzieren.
- **OpenAI**:  
  - API-Key einrichten, in n8n hinterlegen.  
  - In den OpenAI-Nodes das Modell (z. B. GPT-4) und die Sprache (für Whisper) einstellen.
- **Google Sheets**:  
  - Service-Account oder OAuth anlegen, Tabellen-ID herausfinden, in n8n-Credentials eintragen.  
  - Prüfen, ob die in den Nodes verwendete Sheet-ID korrekt ist und ob du Schreib-/Lesezugriff hast.
- **Trello**:  
  - Developer-Key und Token in n8n hinterlegen.  
  - Listen-IDs über Trello-URL ermitteln und in den Nodes („Kurse“, „Mail“ etc.) einstellen.

---

## 7. Testen des Workflows

1. **Workflow aktivieren**.  
2. Im Telegram-Kanal eine **Nachricht** oder eine **Voice Message** an den Bot senden.  
3. Prüfen, ob der Workflow getriggert wird (in n8n → „Executions“).  
4. Schauen, ob eine Trello-Karte in der richtigen Liste angelegt wird (abhängig vom Keyword).  
5. In Google Sheets kontrollieren, ob neue Zeilen mit dem generierten Inhalt auftauchen.  
6. Bei Textproblemen oder falschen Formatierungen in den Code-Nodes nachjustieren.

---

## 8. Fazit

Dieser Workflow zeigt, wie man **n8n** mit **Telegram**, **OpenAI**, **Google Sheets** und **Trello** verknüpft, um:

- **Voice-/Text-Eingaben** aus Telegram KI-basiert zu verarbeiten,  
- **Mehrere KI-Schritte** (Agents) nacheinander ablaufen zu lassen,  
- Die **Ergebnisse** in Google Sheets zu speichern und  
- **Automatisch** passende **Trello-Karten** anzulegen (je nach erkanntem Keyword).

Die wesentlichen Punkte sind das **Einrichten der Credentials**, das **Verknüpfen** der Nodes und das **passgenaue Formatieren** und **Filtern** des KI-Outputs. Mit dieser Anleitung kannst du den Workflow nachbauen, anpassen und in deiner n8n-Instanz ausführen. Viel Erfolg!
